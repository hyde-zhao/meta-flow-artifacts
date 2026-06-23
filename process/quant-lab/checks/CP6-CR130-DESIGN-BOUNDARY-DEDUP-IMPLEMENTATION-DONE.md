---
checkpoint: "CP6"
cr_id: "CR-130"
title: "Design Boundary Dedup Implementation Evidence"
status: "PASS"
created_at: "2026-06-23T17:32:00+08:00"
owner: "host-orchestrator"
context_ref: "process/context/CP6-CR130.context.json"
summary_ref: "process/changes/summaries/CR-130.summary.json"
authorization: "design docs boundary, lightweight checker/tests and process evidence only"
---

# CP6 - CR130 Design Boundary Dedup Implementation Evidence

## Entry Criteria

| Item | Result | Evidence |
|---|---|---|
| CR129 closed | PASS | `process/changes/summaries/CR-129.summary.json` shows closed/ready |
| CR130 active and authorized | PASS | `process/changes/CR-130-DESIGN-BOUNDARY-DEDUP-RUNNER-ARCHITECTURE-AUTHORITY-2026-06-23.md` |
| Scope excludes external permission domains | PASS | CR130 `## 不授权范围` |
| Machine context exists | PASS | `process/context/CP6-CR130.context.json` |

## Implemented Objects

| Object | Action | Notes |
|---|---|---|
| `process/docs/design/HLD.md` | updated | v1.2 / CR130; adds `RUNNER-CORE-MVP-DESIGN-CR126.md` and `strategy-runner-core/DESIGN.md` as current sources; adds Runner Architecture Authority section. |
| `process/docs/design/ARCHITECTURE-DECISION.md` | updated | v1.3 / CR130; marks CR046 as legacy cross-target ADR cluster and adds Runner Core Authority row. |
| `process/docs/design/HLD-CR046-QMT-MINIQMT-DUAL-TARGET-FRAMEWORK.md` | updated | Adds legacy marker, authority pointer and Strategy Runner Core boundary section. |
| `process/docs/design/ARCHITECTURE-DECISION-CR046.md` | updated | Adds legacy ADR marker, authority transfer section and limits ADR-CR046-003 to legacy install design. |
| `process/docs/design/RUNNER-CORE-MVP-DESIGN-CR126.md` | updated | Marks document as CR128 implementation intake/source design and points long-term authority to `strategy-runner-core`. |
| `scripts/check_design_runner_boundary.py` | added | Checks HLD/ADR index, CR046 legacy markers and CR126 source-design marker. |
| `tests/test_cr130_design_runner_boundary.py` | added | Covers current docs pass, CR046-as-current-HLD failure and CLI JSON output. |

## Contract Mapping

| CR130 Acceptance Item | Implementation Evidence |
|---|---|
| HLD current index no longer treats CR046 as current runner authority | `HLD.md` Runner Architecture Authority section and checker. |
| ADR current index marks CR046 as legacy and adds runner authority | `ARCHITECTURE-DECISION.md` Runner Core Authority row and checker. |
| CR046 HLD/ADR include legacy markers | Frontmatter and checker. |
| CR126 design is source design, not long-term authority | Frontmatter and `## 文档定位`. |
| Lightweight checker/test exists | `scripts/check_design_runner_boundary.py`, `tests/test_cr130_design_runner_boundary.py`. |

## Non-Authorized Boundary

CR130 did not authorize or execute:

- `.env`, token, secret, credential, account, funds, positions, orders, fills, session, cookie or raw log reads.
- NAS mount/list/copy/read/write/publish/pull/delete.
- QMT, MiniQMT, XtQuant or gateway runtime startup/connection.
- Provider fetch, lake write, catalog publish or current pointer modification.
- submit/cancel/buy/sell, simulation or live trading.
- `.gitignore` changes, commit, push or Git remote write.

## Exit Criteria

| Item | Result | Evidence |
|---|---|---|
| Design authority docs updated | PASS | Implemented objects table |
| CR046 legacy HLD/ADR markers present | PASS | `scripts/check_design_runner_boundary.py` |
| CR126 source-design marker present | PASS | `scripts/check_design_runner_boundary.py` |
| Guardrail implemented with tests | PASS | `tests/test_cr130_design_runner_boundary.py` |

## Decision

CP6 result: PASS. CR130 is ready for CP7 verification.
