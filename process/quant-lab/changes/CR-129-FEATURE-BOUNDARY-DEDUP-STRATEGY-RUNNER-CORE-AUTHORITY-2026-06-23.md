---
cr_id: "CR-129"
title: "Feature Boundary Dedup: Strategy Runner Core Authority"
cr_type: "process"
cr_kind: "implementation-gate"
lifecycle_status: "closed"
readiness_status: "ready"
gate_status: "closed"
gate_profile: "compact"
status: "closed-current-delivery"
impact_level: "low"
workflow_mode_before: "standard"
workflow_mode_after_change: "standard"
rollback_to: "pre-CR129 feature docs and guardrail state"
approval_result: "user-authorized-feature-boundary-dedup"
created_at: "2026-06-23T16:45:00+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-23T16:45:00+08:00"
source: "user-request"
parent_cr: "CR-128"
source_checkpoint: "process/checks/CP7-CR128-RUNNER-CORE-MVP-VERIFICATION-DONE.md"
source_decision_id: "USER-20260623-START-CR129"
follow_up_type: "feature-boundary-dedup"
risk_class: "docs-and-lightweight-guardrail-no-runtime"
owner: "host-orchestrator"
acceptance_criteria: "CR102/CR103/CR104 empty feature directories removed; qmt-miniqmt-dual-target-framework retained as legacy cross-target framework and points offline runner implementation authority to strategy-runner-core; adjacent feature docs include explicit strategy-runner-core boundary notes; lightweight check prevents cr102/cr103/cr104 runtime validation fragments from reappearing as default feature entries; checks pass; no runtime/NAS/QMT/credential/provider/lake/catalog/trading/Git remote write is performed."
close_condition: "CP6/CP7 evidence confirms feature docs dedup, guardrail test passes and CR129 status closes ready."
conflict_keys: ["strategy-runner-core-authority", "feature-doc-boundary", "cr102-cr103-cr104-default-feature-surface"]
impact_surface: ["process/docs/features", "scripts", "tests", "process/checks", "process/context"]
authz_policy_refs: ["NO_CREDENTIAL_READ", "NO_RUNTIME", "NO_NAS", "NO_TRADING", "NO_REMOTE_WRITE", "NO_PROVIDER_LAKE_CATALOG"]
risk_refs: []
cr_index_path: "process/changes/CR-INDEX.yaml"
historical_baseline_status: "closed"
---

# CR-129 Feature Boundary Dedup: Strategy Runner Core Authority

## 变更描述

用户授权在 CR128 关闭后启动一个小 CR，专门收口 feature 目录中与 `strategy-runner-core` 重叠的边界。目标不是重新设计 runner，而是把长期 feature 入口的职责归属写清楚，并加一个轻量检查防止 CR102/CR103/CR104 运行时验证碎片重新进入默认 feature 面。

授权范围只覆盖 feature 文档边界、空目录清理、轻量检查脚本 / 测试和过程证据；不授权 runtime、NAS、QMT、凭据、provider、lake、catalog、trading 或 Git remote write。

## 文档处理决策

| 受影响文档 / 目录 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `process/docs/features/cr102-nas-real-package-exchange-authorization/` | 删除空目录 | 正式 CR102、CR128、release context 保留历史 | 本 CR | approved |
| `process/docs/features/cr103-qmt-miniqmt-non-trading-day-validation/` | 删除空目录 | 正式 CR103、CR128、release context 保留历史 | 本 CR | approved |
| `process/docs/features/cr104-qmt-miniqmt-trading-day-validation/` | 删除空目录 | 正式 CR104、CR128、release context 保留历史 | 本 CR | approved |
| `process/docs/features/qmt-miniqmt-dual-target-framework/DESIGN.md` | 原文档更新 | 保留为 legacy cross-target framework，不再作为 offline runner implementation authority | 文档内 `## 修订记录` | approved |
| `process/docs/features/execution-semantics-reference/DESIGN.md` | 原文档更新 | 增加与 `strategy-runner-core` 的边界说明 | 文档内 `## 修订记录` | approved |
| `process/docs/features/factor-research-loop/DESIGN.md` | 原文档更新 | 增加与 `strategy-runner-core` 的边界说明 | 文档内 `## 修订记录` | approved |
| `process/docs/features/qmt-gateway-readonly/DESIGN.md` | 原文档更新 | 增加与 `strategy-runner-core` 的边界说明 | 文档内 `## 修订记录` | approved |
| `process/docs/features/qmt-trading-governance/DESIGN.md` | 原文档更新 | 增加与 `strategy-runner-core` 的边界说明 | 文档内 `## 修订记录` | approved |
| `process/docs/features/runtime-authorization-safety/DESIGN.md` | 原文档更新 | 增加与 `strategy-runner-core` 的边界说明 | 文档内 `## 修订记录` | approved |
| `scripts/check_feature_runner_boundary.py` | 新增 | N/A | CP6/CP7 证据 | approved |
| `tests/test_cr129_feature_runner_boundary.py` | 新增 | N/A | CP6/CP7 证据 | approved |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|---|---|---|---|---|
| 需求层 | 是否改变 runner core 功能需求 | feature 文档权威边界 | false | 只收敛文档归属，不改 CR128 runner 行为。 |
| 场景层 | 是否新增运行或交易场景 | CR102/103/104 历史 runtime 场景 | false | 保留历史追溯，但从默认 feature surface 移除。 |
| 计划层 | 是否改变后续 CR 路由 | CR129+ runtime/NAS/QMT 门禁 | true | 明确 runtime/NAS/QMT/provider/lake/catalog/trading 必须另起授权 CR。 |
| 安全层 | 是否触发外部权限 | runtime/NAS/QMT/credential/provider/lake/catalog/trading | false | 全部继续未授权；轻量检查仅扫描本地文档路径。 |
| 交付层 | 是否新增可验证资产 | guardrail script/test/process evidence | true | 新增 feature boundary checker 和 CR129 CP6/CP7 证据。 |

## 实现范围

| Slice | 输出 | 验证 |
|---|---|---|
| S1 Directory cleanup | 删除 CR102/CR103/CR104 空目录 | `find` / checker |
| S2 Legacy boundary | `qmt-miniqmt-dual-target-framework` 增加 retained legacy / superseded_by 语义 | 文档审查 / checker |
| S3 Adjacent boundaries | 五个相邻 feature 增加 “与 strategy-runner-core 的边界” | 文档审查 / checker |
| S4 Guardrail | `scripts/check_feature_runner_boundary.py` 与测试 | pytest / script |
| S5 Evidence | CR129 summary/context/CP6/CP7 | meta-flow checks |

## 不授权范围

- 不读取 `.env`、token、secret、账号、账户、资金、持仓、委托、成交、session、cookie 或原始日志。
- 不访问、列取、挂载、读取、复制、写入、发布、pull 或删除真实 NAS。
- 不启动、连接、安装或运行 QMT / MiniQMT / XtQuant / gateway runtime。
- 不执行 provider fetch、lake write、catalog publish 或 current pointer 修改。
- 不执行 submit/cancel、buy/sell、simulation/live。
- 不修改 `.gitignore`。
- 不执行 commit、push 或 remote write。

## 处理结论

- 当前状态：closed-current-delivery / ready。
- 用户已授权：feature 文档边界、空目录清理、轻量检查脚本 / 测试和过程证据。
- 用户未授权：runtime/NAS/QMT/凭据/provider/lake/catalog/trading/Git remote write。
- 关闭证据：`process/checks/CP6-CR129-FEATURE-BOUNDARY-DEDUP-IMPLEMENTATION-DONE.md`、`process/checks/CP7-CR129-FEATURE-BOUNDARY-DEDUP-VERIFICATION-DONE.md`、`process/archive/CR-129/evidence-index.json`。
