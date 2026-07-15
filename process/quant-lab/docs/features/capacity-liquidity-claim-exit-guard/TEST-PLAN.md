---
status: "ready-for-cp5-review"
version: "1.0"
cr_id: "CR-169"
feature_id: "FEAT-169-04"
---

# FEAT-169-04 测试计划

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator inline meta-dev | 定义 fixture、QAC、claim、CR155 与 Stage2 7/7 验证矩阵。 |

| Test ID | 覆盖 | 通过值 |
|---|---|---|
| CV-T01 | fixture families | 2/2。 |
| CV-T02 | requirements/scenarios/QAC | 9/9、17/17、15/15。 |
| CV-T03 | P0 matrix | 12/12 fail-closed；false PASS=0。 |
| CV-T04 | determinism | 10 runs → 1 hash。 |
| CV-T05 | joint/absent routes | joint fixture pass=1；CR168 absent regression=1。 |
| CV-T06 | forbidden source/ops | canonical/CR168 adapter/aggregate modifications=0；外部操作=0。 |
| CV-T07 | CR155 | status=BLOCKED；paper_candidate=false；promotion=0。 |
| CV-T08 | Stage2 exit shape | contracts=7；每项 status/evidence 完整；7/7 才总 PASS。 |
| CV-T09 | Stage3/real claims | 全 false/0。 |

repository full suite 若失败必须逐项归因；无法证明为既有问题则不得通过 CP7。
