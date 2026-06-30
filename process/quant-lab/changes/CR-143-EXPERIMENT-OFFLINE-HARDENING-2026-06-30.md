---
id: "CR-143"
title: "Experiment Offline Hardening and Handoff"
status: "closed-current-delivery"
kind: "requirement-change"
lifecycle_status: "closed"
readiness_status: "ready"
gate_status: "closed"
gate_profile: "compact"
created_at: "2026-06-30T21:10:00+08:00"
created_by: "host-orchestrator"
updated_at: "2026-06-30T21:45:00+08:00"
source_decision_id: "USER-20260630-COMPLETE-EXPERIMENT-OFFLINE-REMEDIATION"
current_requirement_baseline_path: "process/baseline/CURRENT-REQUIREMENT-BASELINE.yaml"
authz_policy_refs:
  - "NO_CREDENTIAL_READ"
  - "NO_RUNTIME"
  - "NO_NAS"
  - "NO_TRADING"
  - "NO_PROVIDER_FETCH"
  - "NO_REAL_LAKE_LINK_OR_MOUNT"
  - "NO_REAL_LAKE_READ_VALIDATION_WITHOUT_SEPARATE_AUTHORIZATION"
  - "NO_LAKE_WRITE"
  - "NO_CATALOG_POINTER_WRITE"
not_authorized:
  - "real_lake_read_validation"
  - "real_lake_link_or_mount"
  - "nas_access"
  - "provider_fetch"
  - "lake_write"
  - "catalog_pointer_write"
  - "qmt_miniqmt_xtquant_gateway_runtime"
  - "simulation_live_or_trading"
priority: 1
---

# CR-143: Experiment Offline Hardening and Handoff

## 背景

CR140 已完成 experiments 目录归位、helper 收敛和 turnover synthetic adapter parity；CR141 已清零 46 条红基线；CR142 已完成剩余 experiment engine convergence 的核心安全切片。用户明确当前目标是先完成 experiment 整改中不需要连接数据湖的全部任务，再到可连接数据湖的 PC 上读取交接文档执行 E4 真实 lake 只读验证。

本 CR 是 E3 非 lake 收尾审计外壳，不连接真实数据湖，不触碰 NAS/provider/runtime/trading。

## 整体目标

- 完成 E2 远端持久化，降低后续核心模块改动的回退成本。
- 将实验 16 的 IC / Rank IC 汇总从 experiment-local 实现收敛为 `engine.factor_statistics` 的通用离线统计合同。
- 为 experiment-local 动态因子建立 factor ownership 判定矩阵，避免研究因子被误提升为 engine core 或生产因子。
- 补充离线 targeted tests，证明 IC synthetic fixture 等价边界和 factor ownership 规则。
- 形成两个交接文档：
  - 数据湖 PC 交接：用于 E4 真实 lake readonly validation。
  - 后续框架交接：多因子策略、机器学习策略、事件驱动策略的后续补齐输入。

## 范围

### In Scope

- `engine/factor_statistics.py` 增加通用 IC / Rank IC 时间序列和汇总函数。
- `experiments/run_experiment_16_momentum_factor.py` 保留原函数名，但内部调用 engine IC 合同。
- `engine/factor_library.py` 增加 factor ownership 判定，不改变 core factor set。
- 测试覆盖 IC golden fixture、低样本、重复值、常数横截面和 factor ownership。
- 交接文档和验证证据写入 `process/context` / `process/checks`。
- Git remote persistence：本轮用户已明确将 E2 纳入目标，允许推送 source/artifact 当前基线与本 CR 结果。

### Out of Scope

- 不执行真实 lake readonly semantic validation；该事项属于 E4。
- 不执行真实 lake link/mount、NAS、provider fetch、lake write、catalog pointer write。
- 不执行 QMT / MiniQMT / xtquant / gateway runtime、simulation、live 或 trading。
- 不把 `momentum_20d`、`volume_ratio_20d`、`volatility_20d`、RSI、MACD 等实验动态因子静默提升为 engine core。
- 不做机器学习策略生产化或事件驱动策略生产化，只写后续交接输入。

## 五维度影响分析

| 维度 | 影响 | 处理 |
|---|---|---|
| 需求层 | 明确 experiment 离线整改完成边界，并为 E4 / 多因子生产框架提供输入。 | 不刷新产品基线；形成后续交接。 |
| 场景层 | 影响实验 16 IC 统计实现路径；不改变 CLI 或报告字段。 | synthetic fixture + targeted pytest + full pytest。 |
| 计划层 | 合并 E3B-1/2/3 中不需要 lake 的最小实现。 | 本 CR 收敛后进入 E4 授权候选。 |
| 安全层 | 仅 source/test/process 和 Git remote persistence；不触 lake/runtime/trading。 | 若需要真实 lake 或 runtime，停止并转 E4/运行授权门。 |
| 交付层 | 增加两个交接文档，供后续 PC 和后续框架规划消费。 | 明确哪些内容属于多因子、ML、事件驱动。 |

## 阶段计划

| Phase | 目标 | 退出条件 |
|---|---|---|
| Phase 0 | E2 远端持久化和启动盘点 | source/artifact 已推送远端，启动 worktree clean。 |
| Phase 1 | IC engine contract 收敛 | 实验 16 调用 engine IC 函数，旧输出 synthetic fixture 等价。 |
| Phase 2 | Factor ownership matrix | 动态实验因子保持 experiment-local，core 前置包含 calculator 可用性。 |
| Phase 3 | 交接文档 | E4 数据湖 PC 交接和后续策略框架交接完成。 |
| Phase 4 | 验证和关闭 | targeted/full pytest、cr-tracking、hygiene 通过，结果推送远端。 |

## 启动记录

| 项 | 结果 |
|---|---|
| source worktree | 启动时 clean；分支 `precheck/ql-rd-000-redesign-baseline` 已先推送远端。 |
| artifact worktree | 启动时 clean；`main` 已先推送远端。 |
| E2 | 已完成第一段远端持久化：source `340a71b`、artifact `603e9ec` 已在远端。 |
| real lake / NAS / provider / runtime | 未授权，不执行。 |

## 实施结果

| 项 | 结果 |
|---|---|
| E2 | source/artifact 当前基线已先推送远端。 |
| source commit | `0eab0ae` |
| IC / Rank IC | `engine.factor_statistics` 增加 `calculate_information_coefficient_timeseries`、`summarize_information_coefficient`、`spearman_rank_correlation`。 |
| 实验 16 | `calculate_ic_timeseries` / `summarize_ic` 保留原函数名，内部调用 engine IC 合同。 |
| factor ownership | `engine.factor_library` 增加 `factor_ownership_matrix`，实验动态因子保持 `experiment_local`。 |
| 交接文档 | `process/context/EXPERIMENT-OFFLINE-TO-REAL-LAKE-HANDOFF-2026-06-30.md`；`process/context/STRATEGY-FRAMEWORK-FOLLOWUP-HANDOFF-2026-06-30.md`。 |
| targeted verification | `20 passed in 4.73s`；`28 passed in 19.85s`。 |
| full pytest | `1483 passed in 56.12s`。 |
| process hygiene | PASS；`unclassified=0`。 |
| cr-tracking | PASS；关闭前 active formal CRs: CR-143。 |
| evidence | `process/checks/CR143-FINAL-VALIDATION-2026-06-30.md`。 |

## 关闭条件

- E3 中不需要连接真实数据湖的 experiment 整改已完成。
- IC / Rank IC 通用统计合同有 synthetic fixture 和边界测试。
- Factor ownership 规则明确，不误升实验动态因子。
- 数据湖 PC 交接文档和后续策略框架交接文档已写入。
- 全量 pytest 通过，process hygiene 和 cr-tracking 通过。
- 本 CR 结果已推送远端，除非 remote 不可用并记录风险。

## 关闭说明

CR143 关闭为 `closed-current-delivery / READY`。本轮只完成离线 experiment 收尾和交接，不声明真实 lake 语义、模拟盘、实盘或生产策略链路已验证。
