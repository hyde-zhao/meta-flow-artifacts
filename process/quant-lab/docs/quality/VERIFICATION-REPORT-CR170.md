---
title: "CR-170 验证报告"
status: "cp8-awaiting-user"
version: "1.0"
cr_id: "CR-170"
validation_mode: "repository-static-contract"
created_at: "2026-07-15T16:12:00+08:00"
owner: "host-orchestrator inline meta-qa"
---

# CR-170 验证报告

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-15 | host-orchestrator inline meta-qa | 完成 S01-S04、端到端合同、全仓失败归因与治理整改；结论 PASS_WITH_RISK。 |

## 1. 验证范围与边界

本报告验证 canonical Gate 1-5 N/A evidence semantics、Gate 6 protected merge/admission tier hardening、public/adapters/claim compatibility 及仓库治理集成。它不验证真实 evidence、Stage 3 runner 接线、aggregate orchestration、独立 verifier lane、CR-155 promotion 或任何 runtime/真实数据能力。

## 2. Story 结果

| Story | 交付 | CP6 | CP7 | 关键结果 |
|---|---|---:|---:|---|
| S01 | 21-unit inventory 与五态合同 | PASS | PASS_WITH_RISK | 21/21、5/5、15/5/1、4/4 boundary；16 tests。 |
| S02 | Gate 1-5 consumer hardening | PASS | PASS_WITH_RISK | Gate=5/5；conditional audit-only 无 floor；61 tests。 |
| S03 | protected merge 与 tier resolver | PASS | PASS_WITH_RISK | T0/T1/T2/T3=NR/BL/BL/NOT_AUTHORIZED；69 tests。 |
| S04 | compatibility、claim、repository closure | PASS | PASS_WITH_RISK | related=91、governance=27、full=2195/0。 |

## 3. 精确覆盖

| 追踪面 | 实际 | 结论 |
|---|---:|---|
| requirements / scenarios / QAC | 9/9 · 20/20 · 15/15 | PASS |
| policy inventory / Gate coverage / five-state | 21/21 · 5/5 · 5/5 | PASS |
| baseline direction | 15 stricter / 5 controlled-widening / 1 preserve | PASS |
| Gate1 masked escape | classifier / claim / final status=3/3 | PASS |
| tier truth table | T0/T1/T2/T3=4/4 | PASS |
| public compatibility / adapter regression | 100% / 2/2 | PASS |
| end-to-end fixture | 1/1，NR→NR→BLOCKED | PASS |
| aggregate/runner/CR155/real/runtime/remote elevation | 0 | PASS as non-claim |

## 4. 执行结果

| 层 | 结果 |
|---|---|
| CR-170 related | 91 passed / 0 failed |
| governance remediation | 27 passed / 0 failed |
| 首次 repository suite | 2188 passed / 6 failed |
| 最终 repository suite | 2195 passed / 0 failed，98.31 秒 |
| py_compile / diff-check / forbidden production diff | PASS / PASS / 0 |

## 5. 失败归因与整改

首次 6 个失败全部由本轮新增资产触发且已整改：design surface 2 项、process artifact hygiene 2 项、test taxonomy/provenance 2 项。专题 HLD 五件套迁入既有设计归档，测试使用领域名并登记 provenance，hygiene 显式识别 CR-170 active assets；没有失败被豁免或归为未知既有问题。

## 6. Claim ceiling

`stage2_complete=true`、`stage3_started=false`、`stage3_entry_ready=false`；canonical Gate 1-5 N/A semantics 与 Gate 6 admission hardening 已交付，但 `current_stage3_runner_integrated=false`、`aggregate_orchestration_implemented=false`、`real_evidence_available=false`、`runtime_ready=false`、`cr155_promoted=false`。

## 7. 阶段决定

**PASS_WITH_RISK。** 功能、兼容、安全边界和 repository suite 无 blocker；唯一组织性剩余风险为 `R-CR170-VERIFIER-INDEPENDENCE`。`R-CR170-RUNNER-GAP` 是明确的未交付边界，必须由独立 Stage 3 Launch/aggregate CR 决策，不能由 CP8 自动解除。
