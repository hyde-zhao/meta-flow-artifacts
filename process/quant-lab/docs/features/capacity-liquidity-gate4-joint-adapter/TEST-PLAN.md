---
status: "ready-for-cp5-review"
version: "1.0"
cr_id: "CR-169"
feature_id: "FEAT-169-03"
---

# FEAT-169-03 测试计划

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator inline meta-dev | 定义 strict 7-key、DI double、postcondition 与禁止修改回归。 |

| Test ID | 场景 | 精确断言 |
|---|---|---|
| G4J-T01 | valid C3+C4 pair | payload keys=7；canonical calls=1；fixture pass max=1。 |
| G4J-T02 | 4 C3 / 3 C4 mapping | 7/7 exact value/ref identity；第 8 key=0。 |
| G4J-T03 | reason escape | field-level/generic 4 pattern families 全拒绝；canonical calls=0。 |
| G4J-T04 | absent/non-present C4 ref | 3/3 blocked；不得用 N/A reason 绕过。 |
| G4J-T05 | header mismatch | 13 single-field cases all blocked before canonical。 |
| G4J-T06 | canonical non-PASS | worse-state preserved；upgrade count=0。 |
| G4J-T07 | callable double unexpected PASS | postcondition violation rejected；只模拟 public return。 |
| G4J-T08 | source guard | canonical/CR168 adapter/aggregate files modifications=0。 |
| G4J-T09 | claim ceiling | aggregate/capacity-scalable/real-ready/Stage3 PASS 全 0。 |
