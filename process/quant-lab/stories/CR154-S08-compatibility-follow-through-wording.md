---
story_id: "CR154-S08-compatibility-follow-through-wording"
change_id: "CR-154"
title: "Compatibility, follow-through hooks and static evidence wording"
status: "dev-ready"
owner: "meta-dev"
feature_design_refs:
  - "process/docs/features/cross-strategy-reliability-gates/DESIGN.md"
  - "process/docs/features/cross-strategy-reliability-gates/TEST-PLAN.md"
  - "process/docs/features/cross-strategy-reliability-gates/TASKS.md"
source_hld: "process/docs/design/HLD-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES.md"
source_adr: "process/docs/design/ARCHITECTURE-DECISION-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES.md"
lld_policy:
  required: true
  required_level: "technical-note"
  status: "approved"
confirmed: true
confirmed_by: "user"
confirmed_at: "2026-07-03T06:58:00+08:00"
depends_on:
  - "CR154-S02-statistical-artifacts-and-trap-severity"
  - "CR154-S03-cross-strategy-cv-governance"
  - "CR154-S04-pit-universe-survivorship-gate"
  - "CR154-S05-capacity-impact-liquidity-contract"
  - "CR154-S06-regime-attribution-reconciliation-slots"
  - "CR154-S07-admission-default-policy-tier-resolution"
implementation_allowed: true
authorization_boundary: "local/static/fixture-only implementation authorized after CP5 approval; no publish, real release execution or runtime"
---

# CR154-S08 Compatibility, Follow-Through Hooks and Static Evidence Wording

## Goal

Close compatibility, CP4/CP5 follow-through hooks and fixture-only wording without expanding CR154 into runtime or real data readiness.

## Scope

- CR151 / CR152 / CR153 adapter compatibility summary.
- MF-GAP-2/6/7 deferred mapping.
- REQ anchor preservation.
- ML-only n/a policy.
- Static evidence and release wording targets.

## Acceptance Criteria

- `MF-GAP-2/6/7 deferred to factor-evaluation follow-up CR` appears in CP5 acceptance mapping.
- Existing UC/REQ anchors from HLD §11 are preserved or product-baseline refresh is routed before implementation.
- ML-only triple-barrier/meta-labeling/feature-importance fields are explicit `n/a-with-reason` for non-ML strategies.
- `feature_design_refs` paths resolve correctly under the current `process` routing mode, including symlink mode and future non-symlink process roots.
- CP7/CP8 wording states fixture-only contract semantics and no production/paper/live/broker/trading readiness.
- CP5 technical note enumerates exact CR154 return/evidence/release artifact targets or marks each N/A.

## File Ownership

| File | Intent |
|---|---|
| `process/returns/CR154-*.return.json` | Future return packet target list only after implementation. |
| `process/evidence/CR154-*.index.json` | Future evidence index target list only after implementation. |
| `process/checks/CP7-CR154-*.result.json` | Future verification wording target list. |
| `process/checkpoints/CP8-CR154-*.md` | Future release-readiness wording target list. |
| `process/release/RELEASE-CONTEXT-CR154.yaml` | Future release context target list. |

## Design Evidence Required For CP5

Technical note covering exact artifact targets, deferred mappings, compatibility wording, no-runtime boundary and follow-up routing.

## 技术说明

> 本 technical-note 是 CR154-S08 的 CP5 设计证据，`confirmed=false`。本 Story 不创建 S08 LLD，不授权源码实现、测试实现、真实 lake/NAS/provider/runtime/broker/feed/store/catalog/registry/publish，也不授权 paper/live/trading readiness。

### 设计依据

| 来源 | 消费内容 |
|---|---|
| `process/docs/design/HLD-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES.md` | CR151/152/153 compatibility、MF-GAP-2/6/7 deferred、REQ anchor policy、ML-only n/a policy、fixture-only release wording。 |
| `process/docs/design/ARCHITECTURE-DECISION-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES.md` | Shared contract + adapters、CR153 universe compatibility、no-runtime boundary、REQ anchor reuse。 |
| `process/docs/features/cross-strategy-reliability-gates/DESIGN.md` | S08 owns FT-CR154-CP5-004 and compatibility/follow-through wording; package consumes summary only. |
| `process/docs/features/cross-strategy-reliability-gates/TEST-PLAN.md` | S08 fixture group must prove MF-GAP-2/6/7 deferred wording, CR151/152/153 compatibility and no overclaim release text. |
| `process/docs/features/cross-strategy-reliability-gates/TASKS.md` | CR154-T08 requires technical note with exact artifact targets and deferred scope mapping. |
| `process/DEVELOPMENT-PLAN-CR154.yaml` | W4 depends on S02-S07 and owns exact evidence/release wording targets. |

### 文件影响

S08 is a wording, evidence-target and follow-through Story. Future implementation may create or update only the artifact families below after CP5 approval:

| Target | S08 Treatment |
|---|---|
| `process/returns/CR154-S08-compatibility-follow-through-wording.return.json` | Future CP6 return packet target for this Story. |
| `process/evidence/CR154-S08-compatibility-follow-through-wording.index.json` | Future CP6 evidence index target for this Story. |
| `process/checks/CP7-CR154-S08-compatibility-follow-through-wording.result.json` | Future CP7 verification result target for S08 wording and compatibility checks. |
| `process/checkpoints/CP8-CR154-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES.md` | Future CP8 release readiness checkpoint target for CR154 batch-level release wording. |
| `process/release/RELEASE-CONTEXT-CR154.yaml` | Future release context target for CR154 summary, residual risks and no-runtime boundary. |
| `docs/release/RELEASE-NOTES-CR154.md` | N/A for first-wave S08 unless host-orchestrator chooses CR-specific release notes during CP8; default release wording is checkpoint/context scoped. |
| `docs/quality/VERIFICATION-REPORT-CR154.md` | N/A for S08 CP5; CP7 may aggregate into quality docs only after verification routing. |

No `engine/*` or `tests/*` source/test implementation is owned by S08. S08 may reference S07/S02-S06 outputs but does not redefine their resolver or gate semantics.

### 接口 / 数据 / 权限变化

- Interface changes: no runtime interface is introduced by this technical note.
- Data changes: no persistent data, catalog pointer, registry entry, feature store, event store or prediction store mutation is introduced.
- Permission changes: none. The authorization boundary remains local/static/fixture-only.
- Evidence routing:
  - return packet and evidence index paths must be exact Story-specific targets, not broad globs, when implementation occurs;
  - CP7/CP8 wording must state fixture-only contract semantics and no production/paper/live/broker/trading readiness;
  - `feature_design_refs` path resolution must be checked under current process routing mode, including symlink mode and future non-symlink process roots. The check is path-resolution only; it must not access external lake/NAS/provider/runtime resources.

### Compatibility Requirements

| Compatibility Surface | Required S08 Wording |
|---|---|
| CR151 statistical admission gate | CR154 consumes CR151 statistical refs and must not weaken CR151 fail-closed behavior. Existing package-visible CR151 conclusions remain historical evidence, not production readiness. |
| CR152 ML strategy gate | CR154 consumes ML refs and preserves that unfinished ML method work remains CR152/follow-up scope. Triple-barrier, meta-labeling and feature importance are explicit `n/a-with-reason` for non-ML strategies. |
| CR153 event-driven gate | CR153 `universe_pit_audit` remains first-wave source/delegated slot; CR154 owns shared PIT release-blocking wording. Deletion of CR153 slot is out of first-wave scope. |
| StrategyAdmissionPackage | Package may display CR154 summary, blocked claims and release wording only. Summary display must not imply paper/live/trading/broker/runtime readiness. |

### Deferred Mapping

The exact phrase below must appear in CP5 acceptance mapping and downstream evidence wording:

```text
MF-GAP-2/6/7 deferred to factor-evaluation follow-up CR
```

Treatment:

| Gap | S08 Policy |
|---|---|
| MF-GAP-2 | Deferred to factor-evaluation follow-up CR; not a CR154 first-wave blocker unless the active release wording explicitly claims the missing factor-evaluation capability. |
| MF-GAP-6 | Deferred to factor-evaluation follow-up CR; CR154 must preserve traceability and avoid false omission. |
| MF-GAP-7 | Deferred to factor-evaluation follow-up CR; CR154 must not relabel it as solved by shared reliability gates. |

### REQ Anchor Preservation

S08 must preserve HLD §11 anchors unless host-orchestrator routes a product-baseline refresh before implementation. The minimum anchors to keep visible in CR154 evidence are:

| Anchor | Preservation Rule |
|---|---|
| UC-58 | Multifactor chain coverage remains mapped to Gate 1, Gate 2, Gate 4 and Gate 6. |
| UC-59 | ML chain coverage remains mapped to Gate 1, Gate 2, Gate 6 and strategy-specific n/a policy. |
| UC-60 | Event chain coverage remains mapped to Event adapter, CR153 compatibility, Gate 2, Gate 3, Gate 5 and no-runtime boundary. |
| REQ-077 / REQ-136 | Liquidity/capacity/impact gaps block capacity or scale-up claims. |
| REQ-079 / REQ-225 | Walk-forward/OOS and purge/embargo split governance remains visible. |
| REQ-080 / REQ-154 | Reports/admission package must output gates, blocked claims and resolution conditions. |
| REQ-181 | Admission package does not authorize QMT/simulation/live and must preserve blocked reasons. |
| REQ-089 / REQ-095 / REQ-235 / REQ-246 | PIT/current-truth/lookahead/production gate blockers remain explicit. |

If CR154-specific REQ IDs are required later, S08 must route that to host-orchestrator as product-baseline refresh before implementation rather than editing requirements in this Story.

### ML-Only N/A Policy

| Field / Method | Non-ML Wording |
|---|---|
| Triple-barrier labeling | `n/a-with-reason: ML-specific method, not applicable to non-ML strategy class` |
| Meta-labeling | `n/a-with-reason: ML-specific method, not applicable to non-ML strategy class` |
| Feature importance | `n/a-with-reason: ML-specific model diagnostic, not applicable to non-ML strategy class` |

For ML strategies, these fields may be active, deferred or n/a with CR152/follow-up owner. For multifactor and event-driven strategies, they must not become CR154 cross-strategy blockers merely because they are ML-only.

### 异常和回退

| Condition | Behavior |
|---|---|
| `feature_design_refs` path cannot resolve under current process routing mode | CP5/CP6 must block S08 evidence routing and ask host-orchestrator to repair context routing; do not infer alternate paths. |
| CR151/152/153 compatibility wording conflicts with S02-S07 LLD outputs | Stop at CP5 merge; host-orchestrator must reconcile the batch before implementation. |
| Any wording implies production/paper/live/broker/trading readiness | Treat as CP7/CP8 blocker; replace with fixture-only/no-runtime wording. |
| Exact future artifact target is not applicable | Mark N/A with reason in return/evidence index rather than using a broad placeholder. |
| User requests real data/runtime verification | Route to separate authorization gate or formal CR; S08 cannot approve it. |

### 测试入口

Future S08 validation remains fixture-only:

- `tests/research/test_strategy_admission_package.py` should verify CR151/CR152/CR153 compatibility wording and no readiness overclaim after S07 summary attachment.
- `tests/research/test_cross_strategy_reliability_gates.py` may verify ML-only `n/a-with-reason` display if the shared gate summary owns the wording field.
- CP7 result target `process/checks/CP7-CR154-S08-compatibility-follow-through-wording.result.json` should assert:
  - MF-GAP-2/6/7 deferred wording is present;
  - REQ anchors are preserved or product-baseline refresh is routed;
  - fixture-only/no-runtime wording is present;
  - future artifact targets are exact or N/A with reason;
  - no paper/live/broker/trading readiness claim appears.

No test implementation is performed in this CP5 design task.

### 风险与重访条件

| Risk | Impact | Mitigation / Revisit |
|---|---|---|
| Broad artifact globs hide missing Story-specific evidence. | CP6/CP7 may pass without traceable Story evidence. | S8 technical note enumerates exact future targets and requires N/A reasons. |
| Compatibility wording silently changes CR151/152/153 historical meaning. | Regression in package-visible behavior. | Keep CR151/152/153 as consumed evidence sources and do not delete old semantics in CR154 first wave. |
| REQ anchors drift after CP5. | Traceability gap. | Preserve HLD §11 anchors or route product-baseline refresh before implementation. |
| Runtime readiness overclaim. | Safety violation. | Repeated fixture-only/no-runtime wording in CP7/CP8 targets; real operations need separate authorization. |

### 偏离记录

- No S08 LLD is created because `lld_policy.required_level=technical-note`.
- S08 does not own `engine/*` or `tests/*` implementation. It owns future evidence/release wording targets only.
- `docs/release/RELEASE-NOTES-CR154.md` and `docs/quality/VERIFICATION-REPORT-CR154.md` are currently N/A for S08 first-wave CP5 unless CP8/CP7 aggregation explicitly routes them.

### OPEN / LCQ

| ID | Type | Item | Status | Owner |
|---|---|---|---|---|
| O-S08-01 | RESOLVED | S08 wording must be reconciled with final S02-S07 LLD field names and CP5 batch artifact names before implementation. | Host-orchestrator CP5 merge adopts `process/checks/CP5-CR154-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES-LLD-BATCH.result.json`, `process/checkpoints/CP5-CR154-CROSS-STRATEGY-PRODUCTION-RELIABILITY-GATES-LLD-BATCH.md` and `process/checks/CP5-CR154-HUMAN-GATE-LAUNCH-MESSAGE.md` as the batch artifact names; S02-S07 field references resolve through the S01 shared envelope and stable gate ids. | host-orchestrator |
| LCQ-S08 | LCQ | No user-facing clarification is required in this design pass. | n/a-with-reason | meta-dev |
