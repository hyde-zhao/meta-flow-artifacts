---
cr_id: "CR-134"
title: "Runner Evidence Index Integration"
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
rollback_to: "pre-CR134 runner evidence index state"
approval_result: "user-authorized-start-cr134"
created_at: "2026-06-23T16:58:43+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-23T16:58:43+08:00"
source: "user-request"
parent_cr: "CR-133"
source_decision_id: "USER-20260623-START-CR134"
follow_up_type: "runner-development"
risk_class: "offline-runner-only-no-runtime"
owner: "host-orchestrator"
acceptance_criteria: "Runner/CLI can emit a lightweight evidence index JSON for successful offline runs; blocked runs do not write misleading pass indexes; index keeps not_authorization=true, qmt_allowed=false and all forbidden counters at 0; no runtime/NAS/QMT/credential/provider/lake/catalog/trading operation is performed."
close_condition: "CR134 tests and CR/state/runner boundary checks pass; CP6/CP7 evidence records implementation and verification."
conflict_keys: ["strategy-runner-core", "runner-evidence-index", "offline-runner-cli"]
impact_surface: ["trading/strategy_runner", "tests", "process/docs/features/strategy-runner-core", "process/checks", "process/context", "process/changes", "process/state"]
authz_policy_refs: ["NO_CREDENTIAL_READ", "NO_RUNTIME", "NO_NAS", "NO_TRADING", "NO_PROVIDER_LAKE_CATALOG"]
risk_refs: []
cr_index_path: "process/changes/CR-INDEX.yaml"
historical_baseline_status: "closed"
---

# CR-134 Runner Evidence Index Integration

## 变更描述

在 CR133 已完成 JSON/YAML RunSpec 文件入口和 offline CLI 后，本 CR 为 Strategy Runner Core 增加轻量 evidence index 输出能力。

目标是让 runner / CLI 在 successful offline run 后输出可被后续 CP6/CP7 或外部审计消费的索引 JSON。索引只引用 `RunResult` 和脱敏 evidence summary，不复制完整 target portfolio、order details、日志、账户、持仓、委托、成交或运行时正文。

本 CR 不授权 runtime、NAS、QMT、凭据、provider、lake、catalog、trading 或真实账户/委托/成交读取。

## 文档处理决策

| 受影响对象 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `trading/strategy_runner/run_spec.py` | 原代码更新 | CR128/CR133 RunSpec 合同保留；新增 evidence index output path | CP6/CP7 证据 | approved |
| `trading/strategy_runner/runner.py` | 原代码更新 | CR128/CR133 runner API 保留；新增 pass-only index writer 调用 | CP6/CP7 证据 | approved |
| `trading/strategy_runner/cli.py` | 原代码更新 | CR133 CLI 保留；新增 index 输出参数 | CP6/CP7 证据 | approved |
| `trading/strategy_runner/evidence_index.py` | 新增 | N/A | CP6/CP7 证据 | approved |
| `tests/test_cr134_runner_evidence_index.py` | 新增 | N/A | CP6/CP7 证据 | approved |
| `process/docs/features/strategy-runner-core/*` | 原文档更新 | v1.0/v1.1 保留修订记录；新增 v1.2 | 文档内修订记录 | approved |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|---|---|---|---|---|
| 需求层 | 是否改变 runner core 目标 | strategy-runner-core | true | 增加 successful run 的 evidence index 输出能力。 |
| 场景层 | 是否增加验证场景 | offline runner fixture | true | 增加 API、RunSpec、CLI、blocked run 不写 index 场景。 |
| 计划层 | 是否影响后续开发 | runner next slices / CP evidence | true | 后续 CR 可消费 runner evidence index。 |
| 安全层 | 是否触发外部权限 | runtime/NAS/QMT/credential/provider/lake/catalog/trading | false | 外部权限 flags 继续 fail-closed；索引只含脱敏摘要。 |
| 交付层 | 是否新增可验证资产 | code/tests/docs/process evidence | true | 新增测试和 CP6/CP7 证据。 |

## fast-lane 判定

| 条件 | 是否命中 | 说明 |
|---|---|---|
| 仅低风险轻量实现 / 文档 / 规则修改 | false | 涉及 runner 输出合同，走 standard-code。 |
| 修改架构、权限、安全边界或平台安装路径 | false | 不新增权限，只延续 deny-default 边界。 |
| 修改外部接口契约、文件所有权或多 Story 依赖 | true | 新增 CLI / RunSpec 输出字段。 |
| 需要 HLD / LLD 才能解释影响 | false | 现有 Strategy Runner Core feature authority 足够。 |
| 是否保持 fast-lane | false | 保持 `workflow_mode_after_change=standard`。 |

## LLD 设计批次门禁

- 是否需要 LLD 设计批次：false
- batch_id：`CR-134-LITE-CODE-GATE`
- 批次范围来源：CR133 handoff + `strategy-runner-core` feature authority
- 开发启动条件：
  - [x] 当前无 active formal CR
  - [x] CR133 closed READY
  - [x] process route health check passed
  - [x] 外部权限域保持不授权

## 执行链路

| 顺序 | 责任角色 | 动作 | 输入 | 输出 | 门控 | 完成后下一步 |
|---|---|---|---|---|---|---|
| 1 | `host-orchestrator` | 创建 CR 并登记 active | handoff / 用户请求 | 本 CR、state/index 更新 | CR 已登记 | 实现 |
| 2 | `host-orchestrator` | 实现 evidence index 输出 | runner code / feature docs | code/tests/docs | CP6 | 验证 |
| 3 | `host-orchestrator` | 验证并收敛 | tests/checks | CP7 / summary / index | CP7 | 关闭 CR |

## 不授权范围

- 不读取 `.env`、token、secret、账号、账户、资金、持仓、委托、成交、session、cookie 或原始日志。
- 不访问、列取、挂载、读取、复制、写入、发布、pull 或删除真实 NAS。
- 不启动、连接、安装或运行 QMT / MiniQMT / XtQuant / gateway runtime。
- 不执行 provider fetch、lake write、catalog publish 或 current pointer 修改。
- 不执行 submit/cancel、buy/sell、simulation/live。

## 处理结论

- 当前状态：closed-current-delivery / ready。
- 关闭证据：`process/checks/CP6-CR134-RUNNER-EVIDENCE-INDEX-IMPLEMENTATION-DONE.md`、`process/checks/CP7-CR134-RUNNER-EVIDENCE-INDEX-VERIFICATION-DONE.md`、`process/archive/CR-134/evidence-index.json`。
