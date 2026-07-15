---
title: "CR-170 测试策略"
status: "cp8-awaiting-user"
version: "1.0"
cr_id: "CR-170"
validation_mode: "repository-static-contract"
created_at: "2026-07-15T16:12:00+08:00"
owner: "host-orchestrator inline meta-qa"
---

# CR-170 测试策略

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-15 | host-orchestrator inline meta-qa | 冻结 canonical Gate 1-5 N/A semantics、Gate 6 admission policy、兼容/claim 与 repository governance 的分层验证及 CP8 退出条件。 |

## 1. 目标与方法

目标是证明 21/21 policy inventory、五态 N/A decision、Gate 1-5 局部消费与 `resolve_admission_policy` tier 边界满足 fail-closed 不变量，同时保留 public callable、CR-168/169 adapters、CR-155 blocked 与 Stage 3/aggregate 非声明边界。方法采用 ISTQB 风险驱动测试、ISO 25010 功能正确性/可靠性/安全性/兼容性/可维护性，以及领域真值表与 worst-state merge 验证；该集合是 CR-170 批准范围，不是未来真实 evidence 的穷尽测试框架。

## 2. 验证层次

| 层次 | 验证对象 | 精确退出条件 |
|---|---|---|
| contract/unit | 21-unit inventory、五态、4/4 boundary、deterministic reason | 21/21、5/5、4/4；direction=15/5/1 |
| Gate consumers | Gate 1-5 applicable mandatory、conditional audit-only N/A、G1 masked escape | Gate=5/5；generic/incomplete PASS=0；Gate1 三层断言=3/3 |
| admission | protected merge、T0/T1/T2/T3 resolver | NR/BL/BL/NOT_AUTHORIZED=4/4；mandatory NR admission PASS=0 |
| compatibility | public callable/Gate IDs、CR168/169 adapters、CR155、runner/aggregate guards | compatibility=100%；adapters=2/2；promotion/integration=0 |
| end-to-end | structured boundary→Gate→merge→T1 | 1/1，结果 NR→NR→BLOCKED |
| repository governance | design surface、test taxonomy/provenance、artifact hygiene | 27/27；unclassified=0；错误设计根文件=0 |
| repository regression | 全量 pytest | 2195 passed / 0 failed |

## 3. 失败与回退策略

任何 mandatory `MISSING`、`GENERIC_REASON_ESCAPE`、`NA_WITH_INCOMPLETE_BOUNDARY` 或 applicable mandatory `NEEDS_REVIEW` 的无条件 PASS 都阻断 CP7。controlled-widening 只能发生于批准的 5 个 reviewable unit，conditional not-applicable audit ref 不得设置 Gate status floor。兼容、provenance、design surface 或 hygiene 失败均视为 CR-170 引入，除非逐项证明既有；无法证明时不得打开 CP8。

## 4. 授权边界

只允许仓库内源码、fixture/static mapping、测试与过程证据；Python 统一通过 `uv`。不读取真实 lake/NAS/provider/credential，不接 Stage 3 runner，不实现 C1-C4 aggregate，不提升 CR-155，不执行 runtime/QMT/trading、publish/deploy/tag/release 或 Git remote write。

## 5. 风险接受

Host inline 自验证不等于 FU-CR161-006 独立 verifier，故最终决策只能是 `PASS_WITH_RISK / READY_WITH_RISK`，并由 CP8 用户显式决定是否接受。当前 runner gap 和 aggregate 缺失是 claim ceiling，不是被测试“通过”的能力。
