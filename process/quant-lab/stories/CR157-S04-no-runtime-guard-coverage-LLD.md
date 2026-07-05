---
story_id: "CR157-S04-no-runtime-guard-coverage"
change_id: "CR-157"
title: "LLD - No-runtime guard coverage"
design_type: "full-lld"
status: "ready-for-cp5-review"
created_at: "2026-07-05T13:30:00+08:00"
created_by: "host-orchestrator-inline"
owner: "meta-dev"
cp5_review_required: true
implementation_allowed: false
source_hld: "docs/design/HLD-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE.md"
source_adr: "docs/design/ARCHITECTURE-DECISION-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE.md"
feature_design_refs:
  - "docs/features/stage-handoff-guardrails/DESIGN.md"
  - "docs/features/stage-handoff-guardrails/TEST-PLAN.md"
  - "docs/features/runtime-authorization-safety/DESIGN.md"
---

# CR157-S04 LLD - No-Runtime Guard Coverage

## 0. 上游设计依据

| Source | Relevant decision |
|---|---|
| HLD | No-runtime guard must cover lake write, provider fetch, catalog publish, QMT operation, simulation/live and credential read, all zero. |
| ADR-CR157-005 | No-runtime guard is a first-class acceptance criterion. |
| `engine/multifactor_contracts.py` | `FORBIDDEN_OPERATION_COUNTERS` is the shared counter taxonomy. |
| S01/S02/S03 LLD | Package, evidence index and handoff must all consume the same forbidden counter semantics. |

## 1. Goal

Make the no-runtime boundary machine-verifiable across package, evidence and handoff paths by covering every forbidden operation family, not only the six examples called out in the HLD text.

## 2. Requirements

| ID | Requirement | Acceptance |
|---|---|---|
| S04-R1 | Use canonical forbidden counter taxonomy. | Tests iterate every `FORBIDDEN_OPERATION_COUNTERS` field from `engine.multifactor_contracts`. |
| S04-R2 | Any nonzero counter blocks package and handoff readiness. | Validator result is `BLOCKED` and names the counter. |
| S04-R3 | Static/import guard prevents runtime dependency creep. | Source scan test rejects forbidden imports/calls for QMT/provider/credential/runtime/trading packages. |
| S04-R4 | Documentation/release wording cannot imply runtime/publish readiness. | S05 uses S04 wording constraints; CP7/CP8 evidence checks the claims. |

## 3. 模块拆分与职责

| Object/function | Responsibility |
|---|---|
| `FORBIDDEN_OPERATION_COUNTERS` | Canonical counter list in `engine.multifactor_contracts`. |
| `STAGE2_FORBIDDEN_COUNTERS` | Stage 2 alias in `engine.mature_multifactor_framework`. |
| `validate_stage2_no_lake(...)` | Shared counter validator. |
| S01/S03 validators | Call no-runtime validation and propagate blocked reasons. |
| S04 tests | Parametrize over every counter and source-scan prohibited imports. |

## 4. 代码结构与文件影响范围

| File | Change |
|---|---|
| `engine/mature_multifactor_framework.py` | Ensure package/handoff validators call `validate_stage2_no_lake` or equivalent shared counter path. |
| `engine/multifactor_contracts.py` | No planned taxonomy change in CR157; it remains source of truth. |
| `tests/research/test_mature_multifactor_framework.py` | Add parametrized forbidden-counter tests and source import scan after CP5 approval. |

## 5. 数据模型与持久化设计

No new persistent data model. Existing `PermissionCounters` / mapping shape is the input:

| Counter family | Source |
|---|---|
| external project clone/install/run | `FORBIDDEN_OPERATION_COUNTERS` |
| source migration/vendor | `FORBIDDEN_OPERATION_COUNTERS` |
| dependency change | `FORBIDDEN_OPERATION_COUNTERS` |
| provider fetch | `FORBIDDEN_OPERATION_COUNTERS` |
| lake write | `FORBIDDEN_OPERATION_COUNTERS` |
| catalog publish / registry write | `FORBIDDEN_OPERATION_COUNTERS` |
| reports overwrite | `FORBIDDEN_OPERATION_COUNTERS` |
| QMT/gateway operation | `FORBIDDEN_OPERATION_COUNTERS` |
| simulation/live | `FORBIDDEN_OPERATION_COUNTERS` |
| account/order operation | `FORBIDDEN_OPERATION_COUNTERS` |
| credential read | `FORBIDDEN_OPERATION_COUNTERS` |

## 6. API / Interface 设计

| Function | Contract |
|---|---|
| `validate_stage2_no_lake(counters)` | Returns PASS only when every canonical forbidden counter is zero. |
| S01 package validator | Blocks package when no-runtime validation blocks. |
| S03 handoff validator | Blocks handoff when no-runtime validation blocks. |

No function in S04 receives credentials, env values, provider clients, lake handles, broker accounts or runtime handles.

## 7. 核心处理流程

1. Normalize counters with existing `_normalize_permission_counters`.
2. Validate all canonical counter keys.
3. For every nonzero counter, create a blocked reason with the exact counter name.
4. Package/handoff validators add the no-runtime reasons to their own result.
5. Source scan test checks forbidden runtime imports remain absent from `engine/mature_multifactor_framework.py`.

## 8. 技术设计细节

- Parametrized tests should import `FORBIDDEN_OPERATION_COUNTERS` to prevent drift.
- Do not duplicate the counter tuple in tests except for expected minimum coverage comments.
- Source scan is a safety net, not the primary guard; counter validation remains the machine contract.
- Do not read `.env` or credentials as part of guard testing.

## 9. 安全与性能设计

| Area | Decision |
|---|---|
| Security | All forbidden counter checks are local integer checks. |
| Runtime | No runtime import or execution is permitted. |
| Performance | O(number of counters). |
| Audit | CP6/CP7 evidence records counter list and negative fixture refs. |

## 10. 测试设计

| Test | Expected |
|---|---|
| every forbidden counter set to `1` one at a time | `validate_stage2_no_lake` returns `BLOCKED`. |
| package validator with `provider_fetch=1` | package `BLOCKED`. |
| handoff validator with `simulation_or_live=1` | handoff `BLOCKED`. |
| all counters zero | no-runtime validation `PASS`. |
| forbidden import source scan | no prohibited imports/calls found. |
| release wording guard input to S05 | no runtime/publish/paper/live/trading readiness wording. |

## 11. 实施步骤

1. Confirm canonical counter tuple is imported from `engine.multifactor_contracts`.
2. Add parametrized counter tests.
3. Add package/handoff propagation tests.
4. Add source-scan test for forbidden runtime dependencies.
5. Feed wording constraints to S05 documentation technical note.

## 12. 风险、难点与预研建议

| Risk | Mitigation |
|---|---|
| Counter taxonomy changes outside CR157. | Tests import canonical tuple dynamically. |
| Source scan becomes brittle. | Keep scan focused on clearly forbidden imports/names, not comments. |
| Documentation says "ready" without boundary. | S05 must use "static evidence ready" or "CP5/CP6 evidence ready", never runtime-ready. |

### 12.1 Clarification Queue

| ID | Status | Question | Resolution |
|---|---|---|---|
| CQ-CR157-S04-001 | RESOLVED | Does CP5 authorize dry-run runtime probes for guard coverage? | No. CP5 only approves design; CP6 tests stay local/static. |

No OPEN or Spike question blocks CP5.

## 13. 回滚与发布策略

- Roll back added tests/validator calls only; no external state changes.
- No publication, runtime enablement or trading path is part of S04.

## 14. Definition of Done

- CP5 approves full counter coverage design.
- CP6 tests cover every canonical forbidden counter.
- CP7 evidence shows package/handoff block on nonzero counters and no forbidden runtime imports.

## 人工确认区

CP5 reviewer should confirm that all forbidden operation families, including external framework/Git remote/catalog/store/registry paths, remain out of scope for CR157 implementation.
