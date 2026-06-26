---
scenario_pack_id: "RUNNER-QMT-SIMULATION-TO-LIVE-SWITCH-DEFERRED-SCENARIO-PACK-2026-06-25"
workflow_id: "RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-2026-06-25"
created_at: "2026-06-25T12:51:18+08:00"
status: "deferred-recorded"
target_id: "DEFERRED-SIMULATION-TO-LIVE-SWITCH"
runtime_authorization_granted: false
small_live_or_live_authorized: false
implementation_allowed: false
---

# Deferred Simulation To Live Switch Scenario Pack

## 1. Scope

This file records P6 deferred scenarios and future decision items for moving from simulation to live. It is not a live implementation plan and does not authorize `small_live` or `live`.

## 2. Scenario Refs

| Scenario | Summary | Current Status |
|---|---|---|
| `scenario:manual-approval-live-switch` | Live switch requires explicit human approval and separate runtime authorization. | deferred |
| `scenario:simulation-stability-window` | Simulation must pass a stability window before live readiness can be discussed. | deferred |
| `scenario:small-live-limited-scope` | Initial live scope, if approved later, should be small-live with strict limits. | deferred |
| `scenario:kill-switch-and-cancel-only-rollback` | Live rollback must support stop-new-orders and cancel-only/manual takeover. | deferred |

## 3. Decision Queue

| ID | Question | Recommendation | Risk | Status |
|---|---|---|---|---|
| `DQ-LIVE-001` | Use independent live gateway profile or reuse simulation gateway? | Use independent live gateway profile, port and env. | Reusing profile may make simulation runner hit live gateway. | deferred |
| `DQ-LIVE-002` | What stable window and approval are required before live? | Require consecutive simulation success, reconciliation pass and unresolved orders = 0 before approval. | Without stability gate, unverified behavior reaches real funds. | deferred |
| `DQ-LIVE-003` | First live scope: small_live or full live? | Only small_live with symbol, side, quantity and notional limits. | Full live starts with too large a blast radius. | deferred |
| `DQ-LIVE-004` | Which live evidence may be saved? | Save only redacted summaries, counts, digests, refs and risk acceptance records. | Raw account, cash, symbol or order reports expand sensitive data exposure. | deferred |

## 4. Hard Boundary

Deferred status prohibits:

- live submit/cancel implementation
- small_live/live runtime route activation
- credential reads
- raw account / order / funding output
- reuse of simulation authorization as live authorization
- treating this scenario pack as approval
