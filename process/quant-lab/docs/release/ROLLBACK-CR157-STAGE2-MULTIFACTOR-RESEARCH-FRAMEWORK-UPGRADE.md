---
change_id: "CR-157"
status: "ready-with-risk-draft"
profile: "compact"
created_at: "2026-07-05T14:20:00+08:00"
---

# Rollback: CR157

## Rollback Target

Return to the pre-CR157 Stage 2 baseline after CR156 closure. The rollback scope is local source/test/docs/process artifacts only.

## Rollback Scope

| Surface | Rollback Action |
|---|---|
| `engine/mature_multifactor_framework.py` | Revert CR157 additions for package refs, evidence item/index, handoff readiness and counter guard integration. |
| `tests/research/test_mature_multifactor_framework_stage2.py` | Revert CR157-specific test cases. |
| `docs/components/MULTIFACTOR-RESEARCH.md` | Remove CR157 wording if the framework slice is rejected. |
| `docs/product/BACKLOG.md` | Restore pre-CR157 deferred adapter wording if needed. |
| `docs/design/QUANT-RESEARCH-PRODUCTION-ROADMAP.md` | Restore pre-CR157 roadmap status. |
| `process/*CR157*` | Archive or mark CR157 process artifacts as superseded; do not delete ledger evidence. |

## Validation After Rollback

Run the Stage 2 and Stage 3 regression subsets that existed before CR157, then run CR tracking and CR index checks. Do not remove historical checkpoint or gate ledger entries; add a supersession event instead.

## Non-Reversible Items

Checkpoint ledger, gate ledger, dispatch ledger and CR history should remain append-only for auditability.
