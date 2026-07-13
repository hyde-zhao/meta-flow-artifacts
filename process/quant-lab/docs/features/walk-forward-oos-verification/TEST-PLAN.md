---
status: "ready-for-cp5-review"
version: "0.1"
cr_id: "CR-166"
feature_id: "FEAT-166-05"
---

# FEAT-166-05 测试计划

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 0.1 | 2026-07-13 | host-orchestrator inline meta-se | 汇总 CR166 11 场景、12 QAC、回归归因与权限验证。 |

## 场景覆盖

| Story/Feature | Scenarios | QAC |
|---|---|---|
| S01 / FEAT-01 | P01、H01 | 04、07、09 |
| S02 / FEAT-02 | P02、N01..N06、A01、E01 | 01、02、03、04、05、06、10 |
| S03 / FEAT-03 | P01、N01、N05、H01 | 01、07 |
| S04 / FEAT-04 | P01、H01、CR155 regression | 08、11、12 |
| S05 / FEAT-05 | 全部 11/11 + repository attribution | 12/12 |

## 必须执行的测试组

- contract/canonical/C1 golden：6 groups。
- validation/adapters：10 groups。
- producer：8 groups。
- projections/regression：8 groups。
- QAC/authorization/claim：12 QAC + Stage flags 4/4。
- full suite：CR166 新路径引入失败=0；CR165 14 项仅按逐项触达归因处理。

## 证据与判定

每次执行记录 command、exit code、test counts、result ref/hash、operation counters 与 failure attribution。PASS 要求 11/11 scenarios 有证据、12/12 QAC 达标、blocker=0、未经逐项批准的 waiver=0。真实数据、runtime 或 external access 证据不被接纳为本 CR 结果。
