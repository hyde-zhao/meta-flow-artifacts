---
status: "ready-for-cp5-review"
version: "0.1"
cr_id: "CR-166"
feature_id: "FEAT-166-04"
---

# FEAT-166-04 测试计划

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-07-13 | host-orchestrator inline meta-se | 定义 3/3 identity/projection、保守状态、legacy compatibility 与 CR155 回归。 |

| Test ID | 场景 | 断言 |
|---|---|---|
| F166-04-T01 | present daily C2 | projection=3/3；ref/hash/availability/reasons 完全一致。 |
| F166-04-T02 | statistical legacy mapping | fold count/metrics/pass rate 与 C2 可重算且 policy threshold 未迁移。 |
| F166-04-T03 | reliability Gate 2 mapping | split/wf/OOS/purge/embargo/leakage fields 完整，Gate owner 不变。 |
| F166-04-T04 | admission package attach | evidence refs/reasons 追加；runtime authorization flags 4/4 不变。 |
| F166-04-T05 | blocked/unavailable/fail/review | 六类状态均无状态提升，OR-pass=0。 |
| F166-04-T06 | identity mismatch/tamper | 三者均 blocked；raw fold recomputation calls=0。 |
| F166-04-T07 | CR155 historical regression | `paper_candidate=false`、blocked 1/1 保持，无 backfill。 |
| F166-04-T08 | event N/A | 不创建 event component，不将 N/A 作为 mandatory PASS。 |

主测试：`tests/research/test_walk_forward_oos_projections.py`；CR155 复用/扩展 `tests/research/test_statistical_evidence_cr155_regression.py`。通过标准 8/8、consumer=3/3、identity difference=0、status improvement=0、CR155 promotion=0。
