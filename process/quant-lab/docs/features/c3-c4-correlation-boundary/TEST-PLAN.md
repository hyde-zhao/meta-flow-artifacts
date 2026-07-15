---
status: "ready-for-cp5-review"
version: "1.0"
cr_id: "CR-169"
feature_id: "FEAT-169-02"
---

# FEAT-169-02 测试计划

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator inline meta-dev | 定义 13 字段 exact-match、identity 分域与 fail-before-canonical 测试。 |

| Test ID | 场景 | 通过值 |
|---|---|---|
| CH-T01 | 13/13 相等 | correlated pair=1；issues=0。 |
| CH-T02 | 每字段单点 mismatch | 13/13 blocked；canonical calls=0。 |
| CH-T03 | missing/extra/blank | 3 类全 blocked；默认填充=0。 |
| CH-T04 | temporal order | 2 个倒置边界全 blocked。 |
| CH-T05 | same body/different identity | component hash 可相等；join PASS=0；envelope hash 必不同。 |
| CH-T06 | component-specific provenance 不同但共同 context 相同 | 不因专属列表差异误阻断；共同 context mismatch 必阻断。 |
