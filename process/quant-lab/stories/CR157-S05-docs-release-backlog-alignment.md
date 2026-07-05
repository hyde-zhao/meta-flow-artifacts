---
story_id: "CR157-S05-docs-release-backlog-alignment"
change_id: "CR-157"
title: "Docs, release wording and backlog alignment"
status: "verified-with-risk"
owner: "meta-dev"
feature_design_refs:
  - "docs/features/research-evidence-traceability/TASKS.md"
  - "docs/features/stage-handoff-guardrails/TASKS.md"
source_hld: "docs/design/HLD-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE.md"
source_adr: "docs/design/ARCHITECTURE-DECISION-STAGE2-MULTIFACTOR-RESEARCH-FRAMEWORK-UPGRADE.md"
lld_policy:
  required: true
  required_level: "technical-note"
  status: "approved"
  approved_at: "2026-07-05T13:31:00+08:00"
  evidence_ref: "process/stories/CR157-S05-docs-release-backlog-alignment.md#技术说明"
depends_on:
  - "CR157-S01-mature-admission-package-builder-contract"
  - "CR157-S02-research-evidence-index-traceability"
  - "CR157-S03-stage2-stage3-handoff-hardening"
  - "CR157-S04-no-runtime-guard-coverage"
implementation_allowed: true
authorization_boundary: "CP5 approved local source/test implementation only; documentation and backlog alignment; no runtime, real data, publish or trading"
---

# CR157-S05 Docs, Release Wording and Backlog Alignment

## Goal

Align Stage 2 product/component/release wording and backlog refs with the CR157 first slice without overclaiming runtime, event adapter, ML adapter, real validation or publish readiness.

## Scope

- Document target list and exact wording constraints for mature admission package, evidence index and handoff.
- Deferred adapter tracking for `DF-CR157-001` event adapter and `DF-CR157-002` ML adapter.
- Release wording guard: fixture/static-only, no true runtime or performance claims.

## Acceptance Criteria

- Deferred event/ML adapters are visible as backlog refs only.
- Documentation does not claim real data validation, production readiness, paper/live/trading readiness, publish readiness or external framework runtime.
- Release notes can point to package/evidence/handoff static evidence without copying full evidence bodies.

## File Ownership

| File | Intent |
|---|---|
| `docs/components/MULTIFACTOR-RESEARCH.md` | Future component wording update, subject to CP5/CP6 approval. |
| `docs/design/QUANT-RESEARCH-PRODUCTION-ROADMAP-2026-07-01.md` | Future roadmap wording update, subject to CP5/CP6 approval. |
| `docs/product/BACKLOG.md` | Future deferred adapter refs, subject to CP5/CP6 approval. |
| `docs/release/RELEASE-NOTES.md` | Future release wording, subject to CP8. |

## Design Evidence Required For CP5

Technical note listing exact doc targets, wording constraints, deferred refs and release claim boundaries.

## 技术说明

### Design Basis

This Story consumes S01-S04 CP5 designs and aligns documentation/backlog/release wording only. It does not authorize implementation, runtime validation, real data access, publish, paper/live trading, provider fetch, QMT/gateway access, credentials, catalog/store/registry writes, external framework execution or Git remote writes.

### File Impact

| File | Planned CP6 change | Boundary |
|---|---|---|
| `docs/components/MULTIFACTOR-RESEARCH.md` | Add CR157 mature admission package, refs-only evidence index and Stage 2/3 handoff wording. | Must say static/fixture-only; no runtime readiness claim. |
| `docs/design/QUANT-RESEARCH-PRODUCTION-ROADMAP-2026-07-01.md` | Add Stage 2 first-slice completion references and deferred adapter notes. | Must keep event/ML adapters deferred. |
| `docs/product/BACKLOG.md` | Add or refresh `DF-CR157-001` event adapter and `DF-CR157-002` ML adapter refs. | Backlog only; no implementation commitment in CR157. |
| `docs/release/RELEASE-NOTES.md` | Add release note draft only after CP7/CP8 evidence exists. | Must not claim publish, live, paper, QMT or external framework readiness. |

### Interface / Data / Permission Contract

- Inputs: approved S01-S04 implementation/verification refs after CP6/CP7, deferred item ids, CP5 decisions and CP8 release readiness result.
- Outputs: documentation deltas and backlog refs only.
- Permissions: all runtime and external operation counters remain zero; no filesystem paths outside docs/process targets are required.
- Evidence rule: release notes may reference `process/evidence/CR157-*.index.json` but must not inline full reports, diffs, TEST-MATRIX bodies or transcripts.

### Exception And Rollback

- If S01-S04 implementation scope changes, S05 wording waits for updated CP5/CP6 evidence instead of guessing.
- If a release sentence can be read as runtime-ready, paper-ready, live-ready, publish-ready or production-ready, replace it with `static Stage 2 design/fixture evidence ready` or remove the sentence.
- Rollback is documentation-only: revert the affected doc sections and leave deferred backlog refs intact with status `deferred`.

### Test / Verification Plan

| Check | Expected |
|---|---|
| Deferred adapter refs | `DF-CR157-001` and `DF-CR157-002` present as backlog/deferred only. |
| Release wording scan | No real validation, runtime, publish, paper/live/trading, QMT/gateway or production readiness claim. |
| Evidence refs | Release/doc text references evidence index paths and does not copy full bodies. |
| Boundary carry-forward | S04 forbidden operation families remain explicitly non-authorized. |

### Risks

| Risk | Mitigation |
|---|---|
| Documentation overclaims maturity. | Use fixed wording constraints and CP8 review. |
| Deferred adapters look approved. | Keep status `deferred`, reason and trigger condition next to each backlog item. |
| Release notes copy too much evidence. | Reference evidence index ids/paths only. |

### Deviations

No deviation from CP4 plan. S05 remains `technical-note`; full LLD is not required because it changes wording/backlog targets and consumes S01-S04 contracts.

## CP5 Design Evidence

- Evidence type: `technical-note`
- Evidence ref: `process/stories/CR157-S05-docs-release-backlog-alignment.md#技术说明`
- Review status: `ready-for-cp5-review`

## CP5 Approval

- Status: `approved`
- Approved at: `2026-07-05T13:31:00+08:00`
- Design evidence: `process/stories/CR157-S05-docs-release-backlog-alignment.md#技术说明`
- Implementation boundary: local source/test/docs only; no real lake/NAS/provider/credential/QMT/runtime/simulation/live/trading/broker/catalog/store/registry/publish/external framework/Git remote operation.
