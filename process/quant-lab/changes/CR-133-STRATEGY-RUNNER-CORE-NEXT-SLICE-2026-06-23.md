---
cr_id: "CR-133"
title: "Strategy Runner Core Next Slice"
cr_type: "feature"
cr_kind: "implementation-gate"
lifecycle_status: "closed"
readiness_status: "ready"
gate_status: "closed"
gate_profile: "standard"
status: "closed-current-delivery"
impact_level: "medium"
workflow_mode_before: "standard"
workflow_mode_after_change: "standard"
rollback_to: "pre-CR133 runner spec/cli state"
approval_result: "user-authorized-enter-runner-development-after-cr132-hygiene"
created_at: "2026-06-23T18:45:00+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-23T18:45:00+08:00"
source: "user-request"
parent_cr: "CR-132"
source_decision_id: "USER-20260623-START-CR133"
follow_up_type: "runner-development"
risk_class: "offline-runner-only-no-runtime"
owner: "host-orchestrator"
acceptance_criteria: "Runner supports JSON/YAML RunSpec file loading, spec-relative output path, an offline-only CLI entry via python -m trading.strategy_runner.cli, fail-closed invalid spec behavior, feature docs update and tests pass. No runtime/NAS/QMT/credential/provider/lake/catalog/trading operation is performed."
close_condition: "CR133 tests and CR/state/feature/design boundary checks pass; CP6/CP7 evidence records implementation and verification."
conflict_keys: ["strategy-runner-core", "run-spec-file", "offline-runner-cli"]
impact_surface: ["trading/strategy_runner", "tests", "process/docs/features/strategy-runner-core", "process/checks", "process/context", "process/changes", "process/state"]
authz_policy_refs: ["NO_CREDENTIAL_READ", "NO_RUNTIME", "NO_NAS", "NO_TRADING", "NO_PROVIDER_LAKE_CATALOG"]
risk_refs: []
cr_index_path: "process/changes/CR-INDEX.yaml"
historical_baseline_status: "closed"
---

# CR-133 Strategy Runner Core Next Slice

## 变更描述

用户确认 CR132 盘点完成后进入 runner 开发。CR133 选择最小实现切片：为 Strategy Runner Core 增加 JSON/YAML `RunSpec` 文件入口和 offline-only CLI。

本 CR 不授权 runtime、NAS、QMT、凭据、provider、lake、catalog、trading 或真实账户/委托/成交读取。CLI 只运行本地 offline strategy package fixture/spec，并保持 `not_authorization=true`、`qmt_allowed=false`。

## 文档处理决策

| 受影响对象 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `trading/strategy_runner/run_spec.py` | 原代码更新 | CR128 RunSpec 合同保留；新增 file/mapping loader | CP6/CP7 证据 | approved |
| `trading/strategy_runner/runner.py` | 原代码更新 | CR128 runner API 保留；新增 spec file helper | CP6/CP7 证据 | approved |
| `trading/strategy_runner/cli.py` | 新增 | N/A | CP6/CP7 证据 | approved |
| `trading/strategy_runner/result.py` | 原代码更新 | RunResult schema 保持不变；输出父目录自动创建 | CP6/CP7 证据 | approved |
| `tests/test_cr133_runner_spec_cli.py` | 新增 | N/A | CP6/CP7 证据 | approved |
| `process/docs/features/strategy-runner-core/*` | 原文档更新 | v1.0 保留修订记录；新增 v1.1 | 文档内修订记录 | approved |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|---|---|---|---|---|
| 需求层 | 是否改变 runner core 目标 | strategy-runner-core | true | 增加 RunSpec file / CLI 作为 offline 操作入口。 |
| 场景层 | 是否增加验证场景 | offline runner fixture | true | 增加 YAML/JSON spec、CLI、invalid spec fail-closed 场景。 |
| 计划层 | 是否影响后续开发 | runner next slices | true | 后续可基于 CLI 做 fixture/evidence index 集成。 |
| 安全层 | 是否触发外部权限 | runtime/NAS/QMT/credential/provider/lake/catalog/trading | false | 外部权限 flags 继续 fail-closed。 |
| 交付层 | 是否新增可验证资产 | code/tests/docs/process evidence | true | 新增 tests 和 CP6/CP7 证据。 |

## 实现摘要

- `RunSpec.from_file()` 支持 `.json`、`.yaml`、`.yml`。
- `RunSpec.from_mapping()` 校验未知字段、bool 字段、schema、offline mode 和全部外部授权 flags。
- 相对 `package_root` / `output_path` 按 spec 文件所在目录解析。
- 新增 `run_strategy_package_from_spec_file()`。
- 新增 `python -m trading.strategy_runner.cli --spec <path> --json`。
- invalid spec 通过 `RunResult.blocked()` fail closed，CLI 返回非 0。

## 不授权范围

- 不读取 `.env`、token、secret、账号、账户、资金、持仓、委托、成交、session、cookie 或原始日志。
- 不访问、列取、挂载、读取、复制、写入、发布、pull 或删除真实 NAS。
- 不启动、连接、安装或运行 QMT / MiniQMT / XtQuant / gateway runtime。
- 不执行 provider fetch、lake write、catalog publish 或 current pointer 修改。
- 不执行 submit/cancel、buy/sell、simulation/live。

## 处理结论

- 当前状态：closed-current-delivery / ready。
- 关闭证据：`process/checks/CP6-CR133-RUNNER-SPEC-CLI-IMPLEMENTATION-DONE.md`、`process/checks/CP7-CR133-RUNNER-SPEC-CLI-VERIFICATION-DONE.md`、`process/archive/CR-133/evidence-index.json`。
