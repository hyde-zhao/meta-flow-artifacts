---
cr_id: "CR-136"
title: "Runner Bundle Schema / Compatibility Validation"
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
rollback_to: "pre-CR136 runner bundle validation state"
approval_result: "approved-cp8-ready-with-risk"
created_at: "2026-06-23T18:34:21+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-23T19:02:11+08:00"
source: "user-request"
parent_cr: "CR-135"
source_decision_id: "USER-20260623-START-CR136"
follow_up_type: "runner-development"
risk_class: "offline-runner-only-no-runtime"
owner: "host-orchestrator"
acceptance_criteria: "Offline runner bundle inspect/replay can validate manifest schema, required files, file hashes, file sizes, run id/status/pass consistency, not_authorization/qmt boundary, and fail closed for missing/corrupt/incompatible bundles; CLI exposes a validation path; no runtime, NAS, QMT, credentials, provider/lake/catalog, trading or Git write is authorized."
close_condition: "CR136 implementation, tests and runner boundary checks pass; CP6/CP7 evidence records valid bundle validation, corrupt bundle fail-closed behavior, CLI validation and deny-default authorization boundaries."
conflict_keys: ["strategy-runner-core", "runner-artifact-bundle", "runner-bundle-validation", "offline-runner-cli"]
impact_surface: ["trading/strategy_runner", "tests", "process/docs/features/strategy-runner-core", "process/checks", "process/context", "process/changes", "process/state"]
authz_policy_refs: ["NO_CREDENTIAL_READ", "NO_RUNTIME", "NO_NAS", "NO_TRADING", "NO_PROVIDER_LAKE_CATALOG"]
risk_refs: ["WARN-CR136-CR-TRACKING"]
cr_index_path: "process/changes/CR-INDEX.yaml"
current_requirement_baseline_path: "process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml"
historical_baseline_status: "closed"
---

# CR-136 Runner Bundle Schema / Compatibility Validation

## 变更描述

在 CR135 已完成 successful offline run artifact bundle 与 inspect/replay 后，本 CR 固化 bundle 格式合同，并让 inspect/replay/CLI 在读取 bundle 前执行 fail-closed 验证。

最小目标：

1. 定义 CR135 bundle manifest 的 required files、schema version、file ref 和授权边界验证规则。
2. 验证 `run-result.json`、`runner-evidence.index.json`、`run-spec.snapshot.json`、`manifest.json` 齐套。
3. 校验 manifest 中记录的 bytes / sha256 与真实文件一致。
4. 校验 run id、status、passed、package_id、not_authorization、qmt_allowed、forbidden_operation_counters 在 manifest、run-result、evidence index 和 spec snapshot 间一致。
5. 对缺文件、坏 JSON、未知或不兼容 schema、hash/size 不一致、状态不一致、授权边界异常 fail closed。
6. CLI 增加 validate bundle 或等价入口；inspect/replay 自动消费同一验证逻辑。
7. 继续保持 offline-only；不授权 runtime、NAS、QMT、credentials、provider、lake、catalog、trading、Git write 或真实账户 / 委托 / 成交读取。

## CP8 Follow-up 来源

| 字段 | 内容 |
|---|---|
| 父级 CR | `CR-135` |
| 来源决策 ID | `USER-20260623-START-CR136` |
| follow-up 类型 | `runner-development` |
| 风险等级 | `offline-runner-only-no-runtime` |
| owner | `host-orchestrator` |
| 重访条件 | CR135 bundle 已能写入和 inspect/replay，但缺少完整 schema / compatibility validation。 |
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
| `trading/strategy_runner/artifact_bundle.py` | 原代码更新 | CR135 bundle writer / inspect / replay 合同保留；新增 validate fail-closed 合同 | CP6/CP7 证据 | approved |
| `trading/strategy_runner/cli.py` | 原代码更新 | CR135 `--inspect-bundle` 保留；新增 bundle validation CLI 入口 | CP6/CP7 证据 | approved |
| `trading/strategy_runner/__init__.py` | 原代码更新 | 既有 public API 保留；新增 validation export | CP6/CP7 证据 | approved |
| `tests/test_cr136_runner_bundle_validation.py` | 新增 | N/A | CP6/CP7 证据 | approved |
| `process/docs/features/strategy-runner-core/*` | 原文档更新 | v1.0-v1.3 保留修订记录；新增 v1.4 | 文档内修订记录 | approved |

## 旧基线映射

| 原基线对象 | 新增 / 修改对象 | 保留策略 | 映射说明 |
|---|---|---|---|
| CR135 artifact bundle manifest | CR136 manifest validation | 原文保留 | CR136 不改变 bundle 文件集合，只增加读取前验证。 |
| CR135 inspect / replay | CR136 validated inspect / replay | 原文保留 | inspect/replay 必须先 validate，坏 bundle fail closed。 |
| CR134 evidence index | CR136 evidence consistency check | 原文保留 | 只检查 schema/status/run_result_ref/授权边界，不读取外部系统。 |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|---|---|---|---|---|
| 需求层 | 是否新增 runner core 能力 | strategy-runner-core | true | 增加 bundle schema / compatibility validation。 |
| 场景层 | 是否改变测试矩阵覆盖范围 | valid/corrupt bundle、CLI validation | true | 新增缺文件、坏 JSON、hash/size/schema/status/auth mismatch 场景。 |
| 计划层 | 是否改变 Phase、Wave、Story / 任务依赖 | runner next slice | true | CR136 作为 CR135 后续 offline runner slice，进入 standard-code 实现与验证。 |
| 安全层 | 是否引入新的高风险动作或权限要求 | runtime/NAS/QMT/credentials/provider/lake/catalog/trading | false | 只读写本地 bundle 文件；外部权限继续 deny-default。 |
| 交付层 | 是否需要重新生成交付物或回归子集 | code/tests/feature docs/process evidence | true | 新增 validation API/CLI、测试、feature docs v1.4、CP6/CP7 证据。 |

## 回退决策

- 影响范围：局部。
- 回退到阶段：`pre-CR136 runner bundle validation state`。
- 需要重新确认的对象：Strategy Runner Core feature docs、artifact bundle validation API / CLI 合同、CR136 tests。

## fast-lane 判定

| 条件 | 是否命中 | 说明 |
|---|---|---|
| 仅低风险轻量实现 / 文档 / 规则修改 | false | 涉及 bundle 读取合同和 CLI 行为，走 standard-code。 |
| 修改架构、权限、安全边界或平台安装路径 | false | 不新增权限，只延续 deny-default 边界。 |
| 修改外部接口契约、文件所有权或多 Story 依赖 | true | 新增 public validation API 与 CLI 参数。 |
| 需要 HLD / LLD 才能解释影响 | false | 现有 Strategy Runner Core feature authority 足够。 |
| 是否保持 fast-lane | false | 保持 `workflow_mode_after_change=standard`。 |

## LLD 设计批次门禁

- 是否需要 LLD 设计批次：false
- batch_id：`CR-136-LITE-CODE-GATE`
- 批次范围来源：CR 影响分析
- 批次内 Story：N/A，本 CR 使用单一 standard-code implementation gate。
- 批次人工确认稿：N/A
- 开发启动条件：
  - [x] CR136 正式 CR 已登记为 active。
  - [x] CR135 bundle writer / inspect / replay 合同可读。
  - [x] 外部权限边界保持 deny-default。

## 执行链路

| 顺序 | 责任角色 | 动作 | 输入 | 输出 | 门控 | 完成后下一步 |
|---|---|---|---|---|---|---|
| 1 | `host-orchestrator` | 创建 CR 并完成冲突预检 | 用户请求 / CR135 state | 本 CR、CR-INDEX、CR ledger、STATE.current.json | CR 已登记 | 准备实现 |
| 2 | `meta-dev` | 实现 bundle validation API / CLI | CR136、CR135 code、runner feature docs | 代码、测试、实现证据 | CP6 | 交回验证 |
| 3 | `meta-qa` | 验证 bundle validation fail-closed 行为和回归 | CR136 代码、测试、evidence | CP7 验证证据 | CP7 | 交回收敛 |
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
- 台账路径：`process/changes/CR-136-FOLLOW-UP-TRACKING-2026-06-23.md`
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
| 关闭时间 | `2026-06-23T19:02:11+08:00` |
| 关闭结论 | `closed-current-delivery / READY_WITH_RISK` |
| 批准来源 | 用户回复“approve,完成推送后，完成交接文档写作”。 |
| 风险接受 | `WARN-CR136-CR-TRACKING` 作为非阻断 warning，后续候选 `FU-CR136-001`。 |
| Git 推送授权 | 用户显式要求完成推送；范围仅限当前已分类的源码与 process artifact 仓库同步。 |
| 非授权边界 | 本批准不授权 runtime/NAS/QMT/credentials/provider/lake/catalog/trading。 |

## 关联对象

| 类型 | 标识 | 说明 |
|---|---|---|
| Parent CR | `CR-135` | Runner execution artifact bundle / replay workflow。 |
| Feature docs | `process/docs/features/strategy-runner-core/*` | Runner core 长期 feature authority。 |
