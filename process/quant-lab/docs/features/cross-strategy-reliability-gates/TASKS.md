---
feature_id: "FEAT-15"
change_id: "CR-170"
baseline_change_id: "CR-154"
status: "ready-for-cp5-review"
version: "0.3"
created_at: "2026-07-03T10:40:00+08:00"
owner: "host-orchestrator"
implementation_allowed: false
---

# Cross-Strategy Reliability Gates Task Plan

## Revision Record

| Version | Date | Author | Change |
|---|---|---|---|
| 0.1 | 2026-07-03 | host-orchestrator | CR-154 初始任务计划。 |
| 0.2 | 2026-07-15 | host-orchestrator（inline meta-se） | CR-170 增量：四个 full-lld Story、四个串行 Wave、21-unit inventory 与 caller/T3/compatibility 任务。 |
| 0.3 | 2026-07-15 | host-orchestrator（inline meta-dev） | CP5 评审补强：S02 增加 conditional audit-only ref/no-floor 契约，S03 精确到 applicable mandatory unit，S04 增加 public 端到端 fixture 任务。 |

## CR-170 CP5 Design Tasks

| Task ID | Story | 确定性任务 | 设计输出 |
|---|---|---|---|
| CR170-T01 | S01 | 创建 21-unit internal policy inventory 与 five-state classifier LLD | 21/21 exact mapping、15/5/1、caller boundary、reason-id contract。 |
| CR170-T02 | S02 | 修改 Gate1-5 consumer 设计并建立 directional regression | 5/5 Gate、Gate1 3/3、generic/incomplete PASS=0。 |
| CR170-T03 | S03 | 保护 existing merge 并最小硬化 admission T0/T1/T2 | merge 1/1、T0-T3 4/4、T3 diff=0。 |
| CR170-T04 | S04 | 创建 compatibility/claim closure 设计 | public 100%、adapter 2/2、CR155/Stage3/real-op 零提升。 |
| CR170-T05 | S04 | 增加 public-callable evidence→admission 端到端 fixture | n_a_boundaries→Gate NR→merge NR→T1 BLOCKED；PASS=0。 |

## CR-170 Merge Order and File Ownership

1. S01 创建私有 policy contract。
2. S02 消费 S01 并成为 `engine/cross_strategy_reliability_gates.py` 第一写入方。
3. S03 在 S02 合并后成为同一 canonical 文件的第二写入方，禁止并行。
4. S04 只新增回归/claim tests，不修改 CR-168/169 adapters 或 aggregate。

所有 Story 均为 `full-lld`。用户已要求不拉起子 Agent，因此 LLD、开发和验证并发上限均为 `1`；CP5 批准前 implementation/test implementation 均为 `false`。

## CP5 Design Tasks

| Task ID | Story | Task | Output |
|---|---|---|---|
| CR154-T01 | S01 | Design shared gate summary, artifact ref and blocked-claim schema. | Full LLD field contract and first runnable fixture plan. |
| CR154-T02 | S02 | Design Gate 1 statistical artifact model, severity mapping and adapter subtasks. | Full LLD with 12 artifact slots and strategy-family subtask table. |
| CR154-T03 | S03 | Design Gate 2 CV governance contract. | Full LLD with walk-forward/OOS/purged-embargo mapping. |
| CR154-T04 | S04 | Design Gate 3 PIT universe contract and CR153 slot lifecycle. | Full LLD with delegated-to-CR154 compatibility semantics. |
| CR154-T05 | S05 | Design Gate 4 capacity/impact/liquidity contract. | Full LLD with controlled impact enum and no-real-TCA claim boundary. |
| CR154-T06 | S06 | Design Gate 5 regime/attribution/reconciliation slots. | Full LLD with slot/status/ref/n/a validation and no-runtime reconciliation boundary. |
| CR154-T07 | S07 | Design Gate 6 admission tier resolver and release wording. | Full LLD with fail-closed unknown profile behavior. |
| CR154-T08 | S08 | Design compatibility and follow-through wording. | Technical note with exact artifact targets and deferred scope mapping. |

## Future Implementation Task Families

These are planning anchors only. They are not authorized by CP4.

| Family | Candidate Files | Verification Entry |
|---|---|---|
| Shared contract | `engine/cross_strategy_reliability_gates.py` | `tests/research/test_cross_strategy_reliability_gates.py` |
| Admission package integration | `engine/strategy_admission_package.py` | `tests/research/test_strategy_admission_package.py` plus CR154 tests |
| Strategy adapters | `engine/strategy_admission_statistical_gate.py`, `engine/ml_strategy_admission_gate.py`, `engine/event_strategy_admission_gate.py` | Existing CR151/152/153 tests plus adapter fixtures |
| Release/process evidence wording | `process/returns/CR154-*.return.json`, `process/evidence/CR154-*.index.json`, CP7/CP8 CR154 artifacts | CP7/CP8 checks after implementation |

## Merge Order

1. S01 shared contract and fixture skeleton.
2. S02-S06 gate-specific policies may be designed in parallel after S01, but implementation touching the shared contract should merge in story-id order unless CP5 assigns separate file sections.
3. S07 admission policy waits for S02-S06 status semantics.
4. S08 compatibility and wording waits for S02-S07.

## Not Authorized

CP4 does not authorize any implementation task. Any real data, runtime, broker, feed, reconciliation, store/catalog/registry, publish or Git remote operation requires a separate explicit authorization gate.
