---
title: "CR-169 测试报告"
status: "cp7-complete-with-risk"
version: "1.0"
cr_id: "CR-169"
created_at: "2026-07-15T10:12:00+08:00"
owner: "host-orchestrator inline meta-qa"
---

# CR-169 测试报告

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-15 | host-orchestrator inline meta-qa | 固化测试资产、精确结果、全仓失败归因与待 CP8 后置复跑。 |

## 1. 测试资产

| 资产 | 重点 | 状态 |
|---|---|---|
| `test_capacity_liquidity_contracts.py` | 13 header、12 reason、hash identity | PASS |
| `test_capacity_liquidity_producer.py` | Decimal calculation、rounding、refs | PASS |
| `test_capacity_liquidity_envelope_compatibility.py` | catalog/envelope/daily-ML | PASS |
| `test_capacity_liquidity_gate4_projection.py` | 7-key、reason guard、postcondition | PASS |
| `test_capacity_liquidity_cr169_qac.py` | 9/17/15/12、2 fixtures、Stage2 checker | PASS |
| `test_capacity_liquidity_claim_regression.py` | CR168/CR155/claim ceiling | PASS |
| `test_mature_multifactor_framework_stage2.py` | CR157 历史六项与 admission/evidence-index | PASS |
| design/provenance governance | canonical surface 与 test provenance | PASS |

## 2. 结果

- S05：13 passed。
- CR-wide targeted：115 passed。
- full suite 首轮：2154 passed、5 failed。
- full suite 整改后：2157 passed、2 failed。
- 正式 Stage2 result：7 PASS、0 FAIL、0 BLOCKED。

## 3. 失败归因

已修复 provenance 1 项和 design surface 2 项。剩余 2 项均由 `test_process_artifact_hygiene.py` 对当前未提交文件的同一分类规则触发；不存在 CR169 计算、合同、Gate4、claim 或回归失败。由于它们尚未在提交后复跑归零，报告状态保持 `PASS_WITH_RISK`。

## 4. 未覆盖项

真实 ADV/liquidity/capacity、真实参数校准、alpha-decay、runtime、canonical global/aggregate 和 Stage3 均按范围排除，不能从 fixture 结果外推。

## 5. 结论

CR-169 的已批准 fixture/static 合同满足精确验收；CP8 后若授权提交，必须再次运行完整 suite，期望值为 0 failure。若仍有 artifact-hygiene 失败，返回 CP7。
