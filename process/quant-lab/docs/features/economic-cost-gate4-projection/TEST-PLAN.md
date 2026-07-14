---
status: "ready-for-cp5-review"
version: "1.1"
cr_id: "CR-168"
feature_id: "FEAT-168-02"
---

# FEAT-168-02 测试计划

## 修订记录

| 版本 | 日期 | 修订人 | 变更要点 |
|---|---|---|---|
| 1.0 | 2026-07-14 | host-orchestrator inline meta-se | 定义 allowlist/denylist、canonical call count、candidate profile、postcondition 与 source-change guard。 |
| 1.1 | 2026-07-14 | host-orchestrator inline meta-dev | CP5 评审整改：固定 B01 的三个 claim ID，并明确 G4-T06/T07 用 keyword-only public-callable 依赖注入的 double。 |

| Test ID | 场景 | 断言 |
|---|---|---|
| G4-T01 | safe absent B01 | payload keys=4；C4 refs/reasons absent；profile=candidate-release；Gate4 BLOCKED 且精确包含 `adv_participation_missing`、`capacity_dollars_missing`、`liquidity_sizing_missing`。 |
| G4-T02 | 8-key B02 parameterized | 8/8 BLOCKED；canonical_invoked=false；canonical calls=0。 |
| G4-T03 | arbitrary mapping extras | 非 allowlist key 不透传；override escape rejected。 |
| G4-T04 | C3 unavailable/blocked | canonical calls=0；outcome非 PASS。 |
| G4-T05 | C4 present/out-of-scope | canonical calls=0；REJECTED_OUT_OF_SCOPE。 |
| G4-T06 | unexpected PASS double | 通过 `gate4_validator=` 注入 public-result fake；adapter reason=`gate4_unexpected_pass`；PASS exposed=0；aggregate calls=0。 |
| G4-T07 | missing-claims double | 通过同一窄 DI fake 返回缺任一精确 C4 claim 的 non-PASS；adapter reason=`gate4_postcondition_violation`；aggregate calls=0。 |
| G4-T08 | private dependency/source guard | `_has_na_reason` runtime import=0；canonical/aggregate diff=0；adapter-external call=0。 |
| G4-T09 | forbidden operations | 任一 nonzero count blocked；external operations actually executed=0。 |

通过标准 9/9；B01/B02=2/2；capacity/aggregate PASS=0。只用 repo-local public result double，不运行真实 runtime/data。
