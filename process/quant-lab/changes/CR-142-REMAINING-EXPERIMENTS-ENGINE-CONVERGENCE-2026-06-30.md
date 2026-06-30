---
id: "CR-142"
title: "Remaining Experiments Engine Convergence"
status: "active-phase0-contract-audit"
kind: "requirement-change"
lifecycle_status: "active"
readiness_status: "not_ready"
gate_status: "cp2_pending"
gate_profile: "standard"
created_at: "2026-06-30T19:20:00+08:00"
created_by: "host-orchestrator"
updated_at: "2026-06-30T19:20:00+08:00"
source_tracking: "process/changes/CR-140-FOLLOW-UP-TRACKING-2026-06-30.md#FU-CR140-002"
parent_cr: "CR-140"
source_decision_id: "USER-20260630-CONTINUE-EXPERIMENT-REMEDIATION"
current_requirement_baseline_path: "process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml"
authz_policy_refs:
  - "NO_CREDENTIAL_READ"
  - "NO_RUNTIME"
  - "NO_NAS"
  - "NO_TRADING"
  - "NO_PROVIDER_FETCH"
  - "NO_REAL_LAKE_LINK_OR_MOUNT"
  - "NO_LAKE_WRITE"
  - "NO_CATALOG_POINTER_WRITE"
  - "NO_REMOTE_WRITE"
not_authorized:
  - "real_lake_read_validation"
  - "real_lake_link_or_mount"
  - "nas_access"
  - "provider_fetch"
  - "lake_write"
  - "catalog_pointer_write"
  - "qmt_miniqmt_xtquant_gateway_runtime"
  - "simulation_live_or_trading"
  - "git_remote_write_or_push"
priority: 2
---

# CR-142: Remaining Experiments Engine Convergence

## 背景

CR140 已完成 experiments 目录归位、helper 收敛和 turnover synthetic adapter parity；CR141 已清零 46 条红基线。剩余 experiment 整改是 `FU-CR140-002`：让 `run_experiment_17_21_factor_suite.py` 和 `run_experiment_23_29_ml_factor_suite.py` 不再维护与 engine 并行漂移的评估栈。

Phase 4 之前被拆分，是因为这不是简单替换函数：engine 已有 `factor_statistics`、`factor_registry`、`factor_library` 和 `factor_calculators`，但实验 17_21/23_29 使用的 stage3 因子包含 RSI、MACD、MA gap、volume change、max drawdown 等 runner-local 计算，不能静默写入 core registry。

## 整体目标

在不接触真实数据湖和 runtime 的前提下，把剩余实验评估逻辑收敛到 engine 的可复用合同上：

- engine 支持“额外 factor definition + calculator registry + direction”的一致扩展合同。
- 实验 17_21/23_29 尽量复用 `engine.factor_statistics` 的 forward return、排序、分组收益和统计实现。
- 保留实验层必要的报告编排、策略回测和 fixture-only 行为。
- 不声明真实 lake 研究语义、QMT admission、simulation/live/trading 已验证。

## 范围

### In Scope

- 审计并修复 engine factor extension contract 缺口。
- 让 runner-local stage3 因子可通过显式 additional definitions / calculators 表达方向和计算来源。
- 将 17_21/23_29 中可复用的评估侧统计迁移到 engine adapter。
- 增加离线 fixture/targeted 测试证明新旧行为等价或明确 accepted diff。
- 全量 pytest 保持通过。

### Out of Scope

- 不执行真实 lake readonly semantic validation；该事项仍属 `FU-CR140-001`。
- 不执行 NAS、provider fetch、lake write、catalog pointer write。
- 不执行 QMT / MiniQMT / xtquant / gateway runtime、simulation、live 或 trading。
- 不执行 Git remote write / push；该事项仍属 `FU-CR140-004`。
- 不把实验私有 runner-local 因子静默提升为长期 core 因子。

## 五维度影响分析

| 维度 | 影响 | 处理 |
|---|---|---|
| 需求层 | 不改变产品需求；延续 CR140 “研究层一致性”目标。 | 不需要产品基线刷新。 |
| 场景层 | 只影响离线实验和研究报告生成。 | 用 fixture 和 targeted pytest 验证。 |
| 计划层 | 承接 CR140 Phase 4 拆分。 | 本 CR Phase 0 先做 engine contract audit。 |
| 安全层 | 仅 source/test/process 变更；不触碰 runtime/真实 lake/remote。 | 若需要真实 lake 或 runtime 立即停止并转人工门禁。 |
| 交付层 | 可能改变实验内部实现路径，但不改变对外 CLI 入口。 | 全量 pytest + script entrypoint guard 验证。 |

## 阶段计划

| Phase | 目标 | 退出条件 |
|---|---|---|
| Phase 0 | 启动登记和 engine extension contract audit | 明确可安全修复的合同缺口、测试切口和不授权边界。 |
| Phase 1 | engine extension contract 修复 | additional definitions / calculators / direction 合同有测试覆盖。 |
| Phase 2 | 17_21 评估侧 engine adapter | targeted experiment 17_21 通过，新旧关键输出无未解释差异。 |
| Phase 3 | 23_29 复用 17_21/engine adapter | targeted experiment 23_29 通过，不新增 ML optional dependency 要求。 |
| Phase 4 | 全量验证和关闭 | 全量 pytest 通过，cr-tracking 和 artifact hygiene PASS。 |

## 启动记录

| 项 | 结果 |
|---|---|
| source worktree | 启动时 clean。 |
| artifact worktree | 启动时 clean。 |
| cr-tracking | 启动前 active formal CRs: none。 |
| remote write | 未授权，不执行。 |
| real lake / NAS / provider / runtime | 未授权，不执行。 |

## 当前技术判断

现有 engine 已有 `extra_entries` 和 `calculator_registry` 扩展点，但 `compute_equity_factor_matrices` 的方向映射仍只从 core definitions 读取。CR142 的最小安全切片是先让 `additional_definitions` 与 `calculator_registry` 一起进入方向合同，再让实验层通过显式 adapter 复用 engine statistics。

## 关闭条件

- engine extension contract 缺口已修复或明确 deferred。
- 17_21/23_29 不再维护不必要的并行评估统计实现；保留的实验逻辑有理由。
- 全量 pytest 通过。
- 不授权范围保持为 0 次执行。
