---
status: "ready-for-cp5-review"
version: "1.0"
cr_id: "CR-169"
feature_id: "FEAT-169-04"
---

# FEAT-169-04 任务清单

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator inline meta-dev | 建立 S05 fixture/QAC/claim/Stage2-exit evidence 任务。 |

| TASK-ID | Story | 动作 | 输出 | 完成准则 |
|---|---|---|---|---|
| CV-K01 | S05 | 创建 2 个 fixture families | `tests/fixtures/capacity_liquidity/` | 2/2 static-only。 |
| CV-K02 | S05 | 创建 QAC/authorization/source-guard tests | `tests/research/test_capacity_liquidity_cr169_qac.py` | 9/9、17/17、15/15、12/12。 |
| CV-K03 | S05 | 创建 CR155 / CR168 regression | `tests/research/test_capacity_liquidity_claim_regression.py` | CR155 promotion=0；C3-only regression=1。 |
| CV-K04 | S05/CP8 | 定义并生成 Stage2 exit result | `process/checks/STAGE2-EXIT-VERIFICATION.result.json` | 7 items + status/evidence；历史缺口有明确 route。 |

CP5 前只定义任务；CP8 结果不是 source implementation 产物。
