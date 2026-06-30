---
id: "CR-141"
title: "Red Baseline Debt Triage and Closure"
status: "active-phase2-wave1-complete"
kind: "requirement-change"
lifecycle_status: "active"
readiness_status: "not_ready"
gate_status: "cp2_pending"
gate_profile: "standard"
created_at: "2026-06-30T17:35:00+08:00"
created_by: "host-orchestrator"
updated_at: "2026-06-30T18:15:00+08:00"
source_tracking: "process/changes/CR-140-FOLLOW-UP-TRACKING-2026-06-30.md#FU-CR140-003"
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
priority: 1
---

# CR-141: Red Baseline Debt Triage and Closure

## 背景

CR140 已完成 experiments 目录归位、helper 收敛和 turnover synthetic adapter parity，但保留了一个明确风险：全量 pytest 仍有 46 条既有红基线失败。CR140 的交付承诺是“不新增失败”，不是“清零红基线”。用户在 2026-06-30 明确要求继续推进 experiments 的所有整改，因此将 `FU-CR140-003` 正式化为本 CR。

本 CR 的作用是把 46 条红基线从“不透明失败集合”变成有归属、有修复顺序、有 deferred 理由的整改面，并优先关闭不会与后续剩余实验 engine 化重复的债。

## 整体目标

从 quant-lab 端到端链路看，实验层是策略研究和回测判断的上游证据源。当前红基线会让后续 experiments 改造无法清楚判断“是本轮改坏了，还是旧债还在”。CR141 的目标是先把这层噪音清掉或归属清楚：能低风险修的立即修；属于剩余实验 engine 化的失败转入 `FU-CR140-002` / 后续 CR；需要真实 lake 或 runtime 的验证不在本轮执行。

## 范围

### In Scope

- 冻结 CR141 启动时的 pytest 红基线。
- 将 46 条失败按来源和处理策略分类。
- 修复低风险的静态、文档、guardrail、测试契约或明显代码债。
- 对与 `experiments/run_experiment_17_21_factor_suite.py`、`experiments/run_experiment_23_29_ml_factor_suite.py` 的 engine 化高度重叠的失败，明确 deferred 到 `FU-CR140-002`，避免重复劳动。
- 每轮修改后做新增失败对比。

### Out of Scope

- 不执行真实 lake readonly semantic validation；该事项仍属 `FU-CR140-001`，需要单独授权。
- 不推进剩余实验 engine 化主体改造；该事项仍属 `FU-CR140-002`，本 CR 只做红基线归属和不重叠修复。
- 不执行 Git remote write / push；该事项仍属 `FU-CR140-004`，需要单独授权。
- 不触发 NAS、provider fetch、lake write、catalog pointer write、QMT / MiniQMT / xtquant / gateway runtime、simulation、live 或 trading。

## 风险边界

| 风险 | 处理方式 |
|---|---|
| 修复红基线时混入剩余实验 engine 化 | Phase 1 先分类；E3 重叠项只登记 deferred，不在本 CR 强修。 |
| 46 条失败计数一致但集合变化 | Phase 0 冻结失败集合；后续每阶段记录新增失败和修复的基线失败。 |
| 低风险 guardrail 修复触碰真实数据或 runtime | 本 CR 只允许离线静态测试和 synthetic fixture；触碰真实 lake/runtime 即停止并转人工门禁。 |
| 本地 commits 未 push 带来成果丢失风险 | 只记录风险，不在本 CR push；用户授权后由 `FU-CR140-004` 处理。 |

## 阶段计划

| Phase | 目标 | 退出条件 |
|---|---|---|
| Phase 0 | 启动登记、worktree 保护、CR141 红基线冻结 | source/artifact dirty 边界清楚；CR tracking PASS；全量 pytest 基线与失败列表已记录。 |
| Phase 1 | 46 条失败分类 | 每条失败有 bucket、owner、处理建议；E3 重叠项明确 deferred。 |
| Phase 2 | 低风险债修复 | 修复不需要真实 lake/runtime/remote 的静态、文档、guardrail 或局部代码债；无新增失败。 |
| Phase 3 | 剩余失败归属确认 | 剩余失败全部有后续 CR / deferred / blocked reason，不再漂浮。 |
| Phase 4 | 收敛验证与关闭准备 | 全量 pytest 无新增失败；已修复项有 targeted 证据；CR tracking PASS。 |

## Phase 0 启动记录

| 项 | 结果 |
|---|---|
| 用户授权 | 用户要求继续推进 experiments 的所有整改；无风险不发起人工门禁。 |
| source worktree | 启动时 clean，仅本地 ahead 7。 |
| artifact worktree | 启动时 clean，仅本地 ahead 9。 |
| remote write | 未授权，不执行。 |
| real lake / NAS / provider / runtime | 未授权，不执行。 |

## Phase 0 / Phase 1 结果

| 项 | 结果 |
|---|---|
| Phase 0 pytest baseline | `46 failed, 1433 passed in 45.46s` |
| 失败集合 | `process/checks/CR141-PHASE0-BASELINE-FAILURES.txt` |
| baseline report | `process/checks/CR141-PHASE0-PYTEST-BASELINE-2026-06-30.md` |
| Phase 1 分类 | `process/checks/CR141-PHASE1-BASELINE-FAILURE-CLASSIFICATION-2026-06-30.md` |
| 分类结论 | 46 条已分 bucket；`test_experiment_17_21` 和 `test_experiment_23_29` 默认 deferred 到 `FU-CR140-002`，除非后续证明只是独立 fixture drift。 |

## Phase 2 Wave 1 结果

| 项 | 结果 |
|---|---|
| source commit | `ba6dec9` |
| artifact commit | `ed1691e` |
| targeted verification | `17 passed in 1.36s` |
| full pytest comparison | `42 failed, 1437 passed in 44.62s` |
| fixed baseline failures | 4 |
| new failures | 0 |
| evidence | `process/checks/CR141-PHASE2-WAVE1-LOW-RISK-FIXES-2026-06-30.md` |

## 关闭条件

- 46 条启动失败全部完成分类。
- 本 CR 修复项不引入新增 pytest 失败。
- 剩余失败不再是无归属红基线，必须明确属于已修复、后续 engine 化、真实 lake readonly 授权、runtime 授权或其他独立 CR。
- 若无法清零，关闭状态只能是 `READY_WITH_RISK`，且风险必须具体到失败 bucket 和后续候选项。
