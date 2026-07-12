---
status: "completed-with-risk"
version: "1.0"
story_id: "ST-EI-002"
validation_mode: "mixed"
verification_result: "PASS_WITH_RISK"
source_implementation: "process/stories/STORY-ST-EI-002-dispatch-attestation-IMPLEMENTATION.md"
created_by: "host-orchestrator"
created_at: "2026-07-12T04:53:00Z"
---

# Verification: ST-EI-002 dispatch attestation

## Scope and decision

`PASS_WITH_RISK`: 21 direct contract tests plus related event-ledger regression prove the repository adapter rejects false D0/profile/receipt/reuse claims. The current Codex surface still does not provide D0 discovery, explicit selector, spawn receipt or reuse receipt; no test mock is reported as runtime proof. Host-only CP7 was explicitly user-authorized, so independent QA is not claimed.

## Traceability

| Contract | Evidence | Result |
|---|---|---|
| D2 != D0 / untrusted discovery | PC-01..03 | PASS |
| selector + request/receipt correlation | PC-04..08 | PASS |
| required/preferred fallback | PC-09..10 | PASS |
| terminal lifecycle / ID graph | PC-11, PC-14..16, `event dispatch-check` | PASS |
| immutable reuse / PC-19 | PC-12..13, PC-19 | PASS |
| PC-18 six freshness triggers | PC-18 six subtests | PASS |
| live custom profile/model proof | capability probe | UNAVAILABLE / risk |

## Commands

- `uv run pytest tests/test_cr046_dispatch_attestation.py -q` — 21 passed, 6 subtests.
- `uv run pytest tests/test_cp_result_event_ledger.py -q` — 36 passed, 2 subtests.
- `uv run meta-flow event dispatch-check --ledger process/state/AGENT-DISPATCH-LEDGER.ndjson --mode silent` — PASS.
- `git diff --check` — PASS.

## Residual risk

`CR046-RISK-PLATFORM-RECEIPT-UNAVAILABLE` and `CR046-RISK-NO-INDEPENDENT-CP7-AGENT` remain CP8 risk inputs. Functional completion may advance the dependency graph, but final delivery cannot claim platform-attested custom profiles and remains capped at `READY_WITH_RISK` unless Conditional-B is actually satisfied with a new verified QA spawn.
