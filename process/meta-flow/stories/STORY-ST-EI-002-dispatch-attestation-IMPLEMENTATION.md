---
status: "completed"
version: "1.0"
story_id: "ST-EI-002"
story_slug: "dispatch-attestation"
feature_id: "FEAT-EI-CORE"
implementation_type: "platform-contract"
source_story: "process/stories/STORY-ST-EI-002-dispatch-attestation.md"
source_design_evidence: "process/stories/STORY-ST-EI-002-dispatch-attestation-LLD.md"
created_by: "host-orchestrator"
created_at: "2026-07-12T04:51:19Z"
---

# Implementation: ST-EI-002 — Dispatch attempt and platform attestation

## Summary

The new `meta_flow.evidence` package separates event, dispatch, attempt, thread and receipt identities. It implements D2 config validation, fresh D0-only discovery, explicit-selector/receipt verification, immutable thread reuse and A-baseline fallback without inventing a platform receipt.

## Contract mapping

| LLD contract | Implementation | Verification |
|---|---|---|
| typed attempt IDs / terminal closure | `evidence.dispatch.DispatchAttempt`, `validate_attempt_graph`, `advance_attempt`; `event dispatch-check` | PC-11, PC-14..16, typed ledger tests |
| D2 never becomes D0 | `load_profile_config`, `classify_discovery` | PC-01..03, PC-17 |
| PC-18 freshness | `needs_reprobe` checks expiry/session/epoch/config/schema/reload | PC-18 6/6 subtests |
| selector/request/receipt match | `verify_spawn` | PC-04..08 |
| immutable reuse / PC-19 | `ThreadRuntimeIdentity`, `admit_reuse` | PC-12..13, PC-19 |
| required/preferred fallback | `decide_profile_fallback` | PC-09..10 |

## Scope and boundary

- Modified: `meta_flow/evidence/{__init__,dispatch,platform_contract}.py`, `meta_flow/state/event_ledger.py`, `tests/test_cr046_dispatch_attestation.py`.
- No new platform API is claimed. The adapter consumes a receipt only if a platform reports one.
- Legacy rows lacking an `event_id` remain readable as legacy; only typed `dispatch` events are warned as self-declared/unverifiable. `dispatch_id` is no longer used as an event-id fallback.
- No credentials, runtime, publish, commit/push, archive mutation, or quant-lab business code access occurred.

## Verification

| Command | Result |
|---|---|
| `uv run pytest tests/test_cr046_dispatch_attestation.py -q` | 21 passed, 6 subtests passed |
| `uv run pytest tests/test_cp_result_event_ledger.py -q` | 36 passed, 2 subtests passed |
| `uv run meta-flow event dispatch-check --ledger process/state/AGENT-DISPATCH-LEDGER.ndjson --mode silent` | PASS |
| `git diff --check` | PASS |

## Remaining risk / handoff

The current Codex tool schema still exposes no profile selector, profile discovery receipt, spawn receipt, or reuse receipt. The adapter therefore correctly classifies current execution as A-baseline unavailable/degraded rather than platform-attested. CP7 must remain `PASS_WITH_RISK` under the user-approved host-inline route. ST-EI-003 consumes typed attempts for final CP result correlation.
