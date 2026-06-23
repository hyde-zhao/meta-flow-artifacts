---
cr_id: "CR-128"
title: "Runner Core MVP Implementation"
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
rollback_to: "pre-CR128 source and process state"
approval_result: "user-authorized-offline-runner-core"
created_at: "2026-06-23T15:35:00+08:00"
created_by: "host-orchestrator"
approved_by: "user"
approved_at: "2026-06-23T15:35:00+08:00"
source: "user-request"
parent_cr: "CR-126"
source_checkpoint: "process/checkpoints/CP8-CR126-DELIVERY-READINESS.md"
source_decision_id: "USER-20260623-START-CR128"
follow_up_type: "runner-core-mvp-implementation"
risk_class: "offline-runner-core-no-runtime"
owner: "host-orchestrator"
acceptance_criteria: "offline runner core exposes RunSpec, RunResult, run_strategy_package API and CLI/script entry; fixture package runs end-to-end; feature docs consolidate runner scope; CR102/103/104 runtime feature fragments are removed from default feature surface; tests pass; no runtime/NAS/QMT/credential/provider/lake/catalog/trading/Git remote write is performed."
close_condition: "CP6/CP7 verification passes and CR128 evidence records source/test/process changes plus non-authorized boundaries."
conflict_keys: ["runner-core", "strategy-runner", "offline-runner", "feature-doc-governance"]
impact_surface: ["trading/strategy_runner", "scripts/check_cr091_strategy_runner_package.py", "tests", "process/docs/features", "process/checks", "process/context"]
authz_policy_refs: ["NO_CREDENTIAL_READ", "NO_RUNTIME", "NO_NAS", "NO_TRADING", "NO_REMOTE_WRITE", "NO_PROVIDER_LAKE_CATALOG"]
risk_refs: []
cr_index_path: "process/changes/CR-INDEX.yaml"
historical_baseline_status: "closed"
---

# CR-128 Runner Core MVP Implementation

## 变更描述

用户授权启动 Runner Core MVP Implementation。授权范围只覆盖 offline runner core 源码、测试和过程证据；不授权 runtime、NAS、QMT、凭据、provider、lake、catalog、trading 或 Git remote write。

本 CR 同时治理 `process/docs/features` 中 CR102、CR103、CR104 等与 runner 相关但本质属于 NAS / runtime authorization 的特性碎片：该合并的合并到新的 offline runner core feature，该删除的从默认 feature 面删除，历史仍由 CR 正文、检查点和 release context 追溯。

## 文档处理决策

| 受影响文档 | 处理方式 | 旧基线保留方式 | 修订记录位置 | 批准状态 |
|---|---|---|---|---|
| `process/docs/features/strategy-runner-core/DESIGN.md` | 新增 | 吸收 CR091/CR101/CR126/CR128 的 offline runner core 设计口径 | 文档内 `## 修订记录` | approved |
| `process/docs/features/strategy-runner-core/TEST-PLAN.md` | 新增 | 从 CR091/CR098/CR126 验收标准收敛 | 文档内 `## 修订记录` | approved |
| `process/docs/features/strategy-runner-core/TASKS.md` | 新增 | 将 RDS-01..S6 收敛为 CR128 内部实现任务 | 文档内 `## 修订记录` | approved |
| `process/docs/features/cr102-nas-real-package-exchange-authorization/*` | 归档/删除默认面 | 正式 CR102、CP8、release context 保留历史 | 本 CR | approved |
| `process/docs/features/cr103-qmt-miniqmt-non-trading-day-validation/VERIFICATION.md` | 归档/删除默认面 | 正式 CR103、CP7/CP8、release context 保留历史 | 本 CR | approved |
| `process/docs/features/cr104-qmt-miniqmt-trading-day-validation/INPUT-CHECKLIST.md` | 归档/删除默认面 | 正式 CR104、CP8、release context 保留历史 | 本 CR | approved |
| `trading/strategy_runner/*` | 原文档/源码更新 | Git diff + CR128 implementation evidence | CP6/CP7 证据 | approved |
| `scripts/check_cr091_strategy_runner_package.py` | 原文档/源码更新 | 改为复用正式 runner API | CP6/CP7 证据 | approved |
| `tests/test_cr128_runner_core_mvp.py` | 新增 | N/A | CP6/CP7 证据 | approved |

## 五维度影响分析

| 维度 | 评估问题 | 受影响对象 | 结论 | 处理动作 |
|------|----------|-----------|------|---------|
| 需求层 | 是否新增、删除或重定义 REQ-* | runner core MVP scope | true | 以 CR126 runner core MVP 设计为输入，不新增外部权限域。 |
| 场景层 | 是否改变测试矩阵覆盖范围 | offline runner happy path / fail-closed / no-runtime | true | 新增 CR128 tests，复跑 CR091/CR098 回归。 |
| 计划层 | 是否改变 Phase、Wave、Story / 任务依赖 | CR128 implementation wave | true | 单 CR 内部 S1..S6 串行实现；不拆多个正式 CR。 |
| 安全层 | 是否引入新的高风险动作或权限要求 | runtime/NAS/QMT/credential/provider/lake/catalog/trading boundary | true | 明确全部外部权限默认 false；实现不得读取 env 或启动 runtime。 |
| 交付层 | 是否需要重新生成交付物或回归子集 | source/tests/process docs | true | 写 CP6/CP7 证据，运行 pytest 和 meta-flow checks。 |

## 实现范围

| Slice | 输出 | 验证 |
|---|---|---|
| S1 Contract | `trading/strategy_runner/run_spec.py`, `result.py` | RunSpec / RunResult schema tests |
| S2 Orchestration | `trading/strategy_runner/runner.py` | fixture package end-to-end |
| S3 Evidence | runner 集成 `build_evidence_summary` | forbidden counters / redaction tests |
| S4 CLI/API | `__init__.py`, `scripts/check_cr091_strategy_runner_package.py` | script JSON output test / smoke |
| S5 Feature governance | `process/docs/features/strategy-runner-core/*`, delete CR103/104 feature fragments | file existence / review evidence |
| S6 Delivery | CP6/CP7/checks/context/evidence | meta-flow checks |

## 不授权范围

- 不读取 `.env`、token、secret、账号、账户、资金、持仓、委托、成交、session、cookie 或原始日志。
- 不访问、列取、挂载、读取、复制、写入、发布、pull 或删除真实 NAS。
- 不启动、连接、安装或运行 QMT / MiniQMT / XtQuant / gateway runtime。
- 不执行 `scripts/collect_cr099_runner_readonly_smoke.py` 或 `scripts/run_cr104_qmt_miniqmt_validation.ps1`。
- 不执行 submit/cancel、buy/sell、simulation/live。
- 不执行 provider fetch、lake write、catalog publish 或 current pointer 修改。
- 不修改 `.gitignore`。
- 不执行 commit、push 或 remote write。

## 处理结论

- 当前状态：closed-current-delivery / ready。
- 用户已授权：offline runner core 源码、测试、过程证据、feature 目录治理。
- 用户未授权：runtime/NAS/QMT/凭据/provider/lake/catalog/trading/Git remote write。
- 关闭证据：`process/checks/CP6-CR128-RUNNER-CORE-MVP-IMPLEMENTATION-DONE.md`、`process/checks/CP7-CR128-RUNNER-CORE-MVP-VERIFICATION-DONE.md`、`process/archive/CR-128/evidence-index.json`。
