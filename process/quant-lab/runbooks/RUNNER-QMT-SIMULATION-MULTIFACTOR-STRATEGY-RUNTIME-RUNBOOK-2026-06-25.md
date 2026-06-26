---
runbook_id: "RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-RUNBOOK-2026-06-25"
workflow_id: "RUNNER-QMT-SIMULATION-MULTIFACTOR-STRATEGY-RUNTIME-2026-06-25"
created_at: "2026-06-25T12:51:18+08:00"
status: "ready"
runtime_authorization_granted: false
small_live_or_live_authorized: false
non_trading_window_completion_ref: "process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-NON-TRADING-WINDOW-COMPLETION-2026-06-25.md"
---

# Runner QMT Simulation Multifactor Strategy Runtime Runbook

## 1. Authorization Boundary

This runbook is an operational checklist for the simulation multifactor runner contract. It is not runtime authorization.

It does not authorize credential reads, raw account reads, raw order report reads, real QMT runtime execution, real submit/cancel, `small_live`, `live`, provider/lake/catalog writes, NAS operations, or remote writes.

## 1.1 Non-Trading Window Frozen Inputs

| Object | Ref | Status |
|---|---|---|
| Formal StrategyAdmissionPackage | `process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-FORMAL-STRATEGY-ADMISSION-PACKAGE-2026-06-25.json` | frozen for offline readiness; not runtime authorization |
| Formal operator spec | `process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-FORMAL-OPERATOR-SPEC-2026-06-25.json` | frozen for offline readiness; no env, account or broker ref |
| Non-trading completion | `process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-NON-TRADING-WINDOW-COMPLETION-2026-06-25.md` | `PASS_WITH_TRADING_WINDOW_TASKS_DEFERRED` |

The offline operator entry is:

```bash
uv run --python 3.11 python scripts/run_qmt_multifactor_simulation_operator.py \
  --mode fixture \
  --spec-json process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-FORMAL-OPERATOR-SPEC-2026-06-25.json \
  --strategy-admission-json process/context/RUNNER-QMT-SIMULATION-MULTIFACTOR-FORMAL-STRATEGY-ADMISSION-PACKAGE-2026-06-25.json \
  --output-dir process/evidence/runner-simulation-formal-2026-06-25/fixture
```

Do not pass `--env-file` and do not use `--mode runtime` outside an explicit trading-window authorization.

## 2. Required Steps

| Step ID | Required Evidence | Pass Condition | Failure Action |
|---|---|---|---|
| `pre_market_check` | P4 pre-run snapshot digest or fixture ref | Pre-run digest exists and is redacted | Stop before target portfolio generation |
| `gateway_health` | P0 runtime identity evidence | Gateway profile is expected `simulation` profile if runtime is later authorized | Stop before runtime interaction |
| `runtime_profile_check` | P0 expected runtime identity contract | `expected_runtime_mode=simulation` and profile is non-empty | Fail closed before gateway call |
| `target_portfolio_generation` | P1 target portfolio evidence | Target portfolio generated or blocked with reason | Do not create order plan |
| `order_plan_review` | P2 order plan evidence | Order plan generated, risk passed, or no-op | Do not execute |
| `simulation_submit_cancel` | P3 execution evidence | Fake / authorized simulation execution result has no unknown state | Stop and mark manual takeover |
| `reconciliation` | P4 reconciliation evidence | `unresolved_count=0` and reconciliation status pass/warn | Block next run |
| `exception_recovery` | P4 kill-switch candidate / recovery gate | Manual takeover action recorded when blocked | Do not resume automation |
| `eod_cancel_unfinished` | P3 cancel-open-orders evidence | Unfinished cancelable refs are planned or cancelled under explicit authorization | Record manual takeover before next run |

## 3. Normal Flow

1. Confirm P0 runtime identity contract is present.
2. Generate P1 target portfolio from multifactor signals.
3. Generate P2 simulation order plan from current position summary and target portfolio.
4. Execute P3 only through a caller-provided simulation gateway interface.
5. Run P4 reconciliation before allowing the next run.
6. Archive only evidence refs, counts, buckets and digests.

## 4. Failure Flow

Any blocked, rejected or unknown execution status stops the next submit batch. The operator action is:

1. Stop new orders.
2. Keep cancel actions planned-only unless explicit simulation runtime authorization is active.
3. Record manual takeover.
4. Rerun reconciliation.
5. Resume only when reconciliation passes and manual takeover is recorded.

## 4.1 Manual Takeover And Kill-Switch

| Trigger | Required Action | Resume Condition |
|---|---|---|
| `session_expired` | Stop runner; do not enter P3; restart / re-login gateway only under fresh authorization. | Health, capabilities and readonly positions pass again. |
| `order_submit_unknown` | Stop new orders; record unknown intent digest; manually inspect MiniQMT. | Unknown state becomes accepted, rejected or cancelled, then P4 passes. |
| `cancel_unknown` | Stop new submit; avoid repeated unknown cancel loops. | Cancel state is explicit and manual takeover is closed. |
| `recon_diff` | Stop next automated run; record diff bucket and evidence ref. | Diff is closed and P4 passes. |
| redaction failure | Discard evidence; do not publish. | Evidence validation passes after rerun. |

Kill-switch rules are fail-closed: missing authorization, active kill switch, gateway health failure, unknown order state, unclosed reconciliation diff, or raw evidence leakage all stop new orders. Cancel actions require explicit simulation runtime authorization.

## 4.2 Stability Window

The daily simulation readiness window is frozen as `5` runs across `3` trading days. Passing criteria are valid per-run authorization, P1-P4 evidence schema pass, `unknown_count=0`, closed manual takeover records and gateway stop confirmation. Non-trading fixture evidence does not count toward this runtime stability window.

## 5. Deferred Live Switch

`small_live` and `live` are out of scope. The deferred live-switch scenario pack is:

`process/context/RUNNER-QMT-SIMULATION-TO-LIVE-SWITCH-DEFERRED-SCENARIO-PACK-2026-06-25.md`

That pack records future decisions. It does not authorize implementation or runtime execution.

## 6. Phase B Simulation Readiness Addendum

Phase B simulation operational readiness closure was recorded on 2026-06-26.

| Item | Status | Evidence |
|---|---|---|
| Stability count gate | `PASS` | `process/evidence/RUNNER-QMT-SIMULATION-MULTIFACTOR-STABILITY-WINDOW-SUMMARY-2026-06-26-r6.json` |
| Completed runs | `5/5` | r6 summary and r3-r6 operator evidence indexes |
| Latest runtime acceptance | `PASS_STABILITY_WINDOW_COMPLETE` | `process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-TRADING-WINDOW-R6-RUNTIME-ACCEPTANCE-2026-06-26.md` |
| Readiness closure | `READY_WITH_RISK` | `process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-SIMULATION-OPERATIONAL-READINESS-CLOSURE-2026-06-26.md` |
| Human risk acceptance | `ACCEPTED` | `process/checkpoints/RUNNER-QMT-SIMULATION-MULTIFACTOR-SIMULATION-READINESS-HUMAN-DECISION-2026-06-26.md` |

Readiness interpretation:

1. The simulation runtime count gate is complete: five authorized runtime runs have passed P0/P0.5/P1/P2/P3/P4 with `unknown_count=0`, `unresolved_count=0`, and no manual takeover required.
2. The owner accepted the runbook's conservative 3-trading-day observation gap as a `READY_WITH_RISK` condition on 2026-06-26. An additional trading-day observation is optional unless a future gate requires upgrading the decision to `READY`.
3. This addendum does not authorize repeated runtime submit/cancel. Further runtime requires a new objective and fresh per-run authorization.
4. This addendum does not authorize `small_live` or `live`; those remain behind a separate CR, human decision, and runtime authorization gate.

## 7. Ops Policy Addendum

`SIM-OPS-GATE-02` and `SIM-OPS-GATE-03` were closed as `POLICY_DEFINED` on 2026-06-26.

| Item | Status | Evidence |
|---|---|---|
| Runtime input policy | `POLICY_DEFINED` | `process/policies/RUNNER-QMT-SIMULATION-MULTIFACTOR-RUNTIME-INPUT-GATEWAY-LIFECYCLE-POLICY-2026-06-26.md` |
| Gateway lifecycle policy | `POLICY_DEFINED` | `process/policies/RUNNER-QMT-SIMULATION-MULTIFACTOR-RUNTIME-INPUT-GATEWAY-LIFECYCLE-POLICY-2026-06-26.md` |
| Ops policy closure | `PASS` | `process/checks/RUNNER-QMT-SIMULATION-MULTIFACTOR-OPS-POLICY-CLOSURE-2026-06-26.md` |

Operational interpretation:

1. Runtime overlay/spec/admission files remain private runtime inputs and must stay outside `process/` and Git tracked paths.
2. Formal evidence can save only redacted summaries, counts, buckets, status, reason codes and path refs.
3. Windows gateway lifecycle remains user-managed. Code changes under `trading/*` that affect the gateway must be synced to `/mnt/c/quant-lab-runtime`, followed by a user restart and fresh P0/P0.5 checks.
4. The documented gateway start command and health checks do not authorize submit/cancel; every runtime still requires a fresh per-run authorization.
