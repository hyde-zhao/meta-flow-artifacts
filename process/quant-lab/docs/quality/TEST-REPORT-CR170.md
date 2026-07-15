---
title: "CR-170 测试报告"
status: "cp8-awaiting-user"
version: "1.0"
cr_id: "CR-170"
created_at: "2026-07-15T16:12:00+08:00"
owner: "host-orchestrator inline meta-qa"
---

# CR-170 测试报告

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-15 | host-orchestrator inline meta-qa | 固化 CR-170 定向、治理、全仓测试结果与 6 个初始失败的完整归因。 |

## 1. 主要测试资产

| 资产 | 重点 | 状态 |
|---|---|---|
| `test_reliability_na_policy.py` | 21-unit、五态、4/4 boundary、15/5/1 | PASS |
| `test_cross_strategy_reliability_gates.py` | Gate 1-5 局部消费、三层断言、conditional audit-only | PASS |
| `test_reliability_admission_policy.py` | merge 保留、T0-T3、T3 兼容 | PASS |
| `test_canonical_reliability_regression.py` | public/adapters/end-to-end/claim ceiling | PASS |
| Meta Flow governance subset | design surface、taxonomy/provenance、hygiene | PASS |

## 2. 精确结果

- S01：16 passed。
- S02 及相邻：61 passed。
- S03 及相邻：69 passed。
- CR-wide related：91 passed。
- governance subset：27 passed。
- repository full suite 首次：2188 passed、6 failed。
- repository full suite 最终：2195 passed、0 failed（98.31 秒）。

## 3. 初始失败归因

| 组 | 数量 | 根因 | 修复 | 最终 |
|---|---:|---|---|---:|
| design surface | 2 | 5 个专题设计副本位于 authority root | 原文迁入 `process/archive/design-cr-docs/`，更新 index/ref | 0 |
| process hygiene | 2 | CR-170 source/Feature 资产未进入 active 分类 | 增加 CR170 scoped 分类与回归 | 0 |
| test taxonomy/provenance | 2 | CR 命名测试文件 + 3 个新测试未登记 | 改领域名、补 CR/Story provenance | 0 |

## 4. 未覆盖项

真实数据/evidence、Stage 3 runner、aggregate、独立 verifier、CR155 promotion、runtime/trading 和远端发布均按范围排除，不能从 2195/0 外推为这些能力已就绪。

## 5. 结论

CR-170 批准范围内测试失败为 `0`，未知归因为 `0`，满足提交 CP8 的自动质量条件。由于验证由 Host inline 执行，终验只能建议 `READY_WITH_RISK`。
