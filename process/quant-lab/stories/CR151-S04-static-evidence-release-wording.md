---
story_id: "CR151-S04-static-evidence-release-wording"
change_id: "CR-151"
title: "Static evidence and release wording"
status: "lld-ready-for-review"
owner: "meta-dev"
feature_design_refs:
  - "docs/features/factor-research-loop/TEST-PLAN.md"
  - "docs/features/factor-research-loop/TASKS.md"
source_hld: "process/docs/design/HLD-CR151-STRATEGY-ADMISSION-STATISTICAL-GATE.md"
source_adr: "process/docs/design/ARCHITECTURE-DECISION-CR151-STRATEGY-ADMISSION-STATISTICAL-GATE.md"
lld_policy:
  required: true
  required_level: "technical-note"
  status: "pending-cp5"
depends_on:
  - "CR151-S01-statistical-report-contracts"
  - "CR151-S02-gate-evaluator-fail-closed-rules"
  - "CR151-S03-admission-completion-linkage"
implementation_allowed: false
authorization_boundary: "static/fixture only; no real lake/NAS/provider/QMT/runtime/simulation/live/trading/broker/credential/external framework/Git remote"
---

# CR151-S04 Static Evidence And Release Wording

## Goal

Make CR151 evidence and release wording unambiguous: validation is static/fixture-only and does not prove real performance or runtime readiness.

## Scope

- CP6 return/evidence shape
- CP7 verification wording
- CP8 release wording
- Deferred Wave B trace rows
- No-real-operation counters

## Acceptance Criteria

- Evidence states `effective_validation_mode=static-only`.
- No real data, NAS, provider, QMT, runtime, trading, broker, credential, external framework or Git remote operation is claimed.
- Deferred Wave B items stay visible in release notes and follow-up tracking.
- CP8 cannot mark real performance readiness from fixture PASS.

## File Ownership

| File | Intent |
|---|---|
| `process/returns/*` | Story return packet shape after implementation. |
| `process/evidence/*` | Evidence index wording after implementation. |
| `docs/release/*` | Release readiness wording after implementation. |

## Design Evidence Required For CP5

Technical note covering evidence fields, release wording, deferred items, validation commands and static-only claim boundary.

## 技术说明

### 设计依据

| 来源 | 消费内容 |
|---|---|
| `process/docs/design/HLD-CR151-STRATEGY-ADMISSION-STATISTICAL-GATE.md` | Flow 3 CP7/CP8 evidence；static-only boundary；Wave B deferred items |
| `process/docs/design/ARCHITECTURE-DECISION-CR151-STRATEGY-ADMISSION-STATISTICAL-GATE.md` | ADR-CR151-004 no-runtime validation boundary |
| `process/docs/design/FEATURE-DESIGN-MATRIX.md` | S04 technical-note policy and CR151 Wave A / Wave B boundary |
| `docs/features/factor-research-loop/TEST-PLAN.md` | CR151 static-only evidence boundary validation |
| `docs/features/factor-research-loop/TASKS.md` | FEAT-03-T01L evidence wording task |

### 文件影响

| 动作 | 文件 / 路径 | 说明 |
|---|---|---|
| 创建 / 更新 | `process/returns/CR151-*` | CP6 Story return packet must record implementation objects, design delta refs and static-only validation mode. |
| 创建 / 更新 | `process/evidence/CR151-*` | CP7 evidence index must record report refs, test refs and no-real-operation counters. |
| 创建 / 更新 | `process/checks/CP7-CR151-*` | CP7 result must state `effective_validation_mode=static-only` and forbidden operations = 0. |
| 创建 / 更新 | `process/checkpoints/CP8-CR151-*` / `docs/release/*` | CP8 release wording must not claim runtime, real performance or trading readiness. |

### 接口 / 数据 / 权限变化

| 项 | 设计 |
|---|---|
| Evidence mode | `effective_validation_mode=static-only` is mandatory in CP7/CP8 evidence. |
| Operation counters | Evidence records zero counts for credential, real lake, NAS, provider, QMT/runtime, broker/trading, external framework, Git remote and catalog pointer operations. |
| Deferred scope | Evidence and release wording keep Wave B deferred items visible: IC decay by lag, IC half-life, turnover, liquidity/capacity view, orthogonalization against known risk factors, monotonicity, quantile spread, regime-aware validation, factor correlation clustering/de-duplication, capacity/impact, IR/TE/Active Share and PIT universe audit. |
| Permissions | No new permissions. S04 cannot authorize runtime, data lake, NAS, credential or trading operations. |

### 异常和回退

| 失败 / 异常 | 行为 |
|---|---|
| Evidence omits static-only mode | CP7/CP8 must block release readiness until wording is fixed. |
| Any forbidden operation counter is nonzero | CR151 release is `NOT_READY` or blocked pending a separate runtime authorization decision. |
| CP8 wording implies real performance readiness | Reopen release wording before approval; do not proceed with misleading release. |
| Wave B item disappears from deferred tracking | Reopen S04 evidence wording and follow-up table before CP8. |

### 测试入口

| 场景 | 验证入口 |
|---|---|
| CP7 evidence mode present | CP7 result/evidence index review |
| No-real-operation counters zero | `tests/test_cr151_strategy_admission_statistical_gate.py` plus CP7 evidence scan |
| CR150 linkage remains static-only | `tests/test_cr150_multifactor_framework_completion.py` |
| Release wording static-only | CP8 release-readiness checklist |

### 已知风险

| 风险 | 缓解 |
|---|---|
| Statistical PASS 被误读为 simulation / paper / live readiness | Release wording explicitly says fixture/static only and runtime remains unauthorized. |
| Deferred Wave B 被读者误以为完成 | Evidence and release notes include explicit deferred rows for IC decay, half-life, turnover, orthogonalization, monotonicity, quantile spread, regime-aware validation and factor correlation clustering/de-duplication. |
| S04 被当作 runtime authorization | Technical note and CP8 checklist state S04 cannot authorize runtime/data/NAS/credential operations. |

### 偏离记录

No deviation from CP3 decisions. S04 remains a technical-note Story because it changes process/evidence wording only and does not define new source contracts beyond S01-S03.
