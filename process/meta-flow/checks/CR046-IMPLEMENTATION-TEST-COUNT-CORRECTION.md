---
correction_id: "CR046-CORR-001"
cr_id: "CR-046"
status: "applied"
created_at: "2026-07-12T05:00:00Z"
supersedes: "conversational implementation progress summary reporting 85 passed / 50 subtests"
scope: "reporting only; no source, CP decision, result hash, or historical command output is rewritten"
---

# CR-046 implementation test-count correction

## Corrected fact

The previous conversational progress summary stated `85 passed, 50 subtests` without naming one exact pytest invocation. That figure was an invalid aggregation of separate command results and is withdrawn.

| Measurement scope | Exact command | Result | Interpretation |
|---|---|---:|---|
| CR-046 focused W1 + W2 | `uv run pytest tests/test_state_transition.py tests/test_cr046_dispatch_attestation.py -q` | `49 passed, 48 subtests passed` | Current focused implementation regression scope. |
| Full repository | `uv run pytest -q` | `377 passed, 70 subtests passed` | Current full-suite result after the ST-EI-003 through ST-EI-007 implementation fixtures present at the time of this correction update. |
| Historical CP records | Exact commands in Story return/evidence packets | Preserved unchanged | Their `27/28`, `21`, and `36` counts are command-specific point-in-time evidence, not additive totals. |

## Impact assessment

- CP6 / CP7 decisions for ST-EI-001 and ST-EI-002 remain unchanged: their individual commands were correctly recorded.
- No code, result JSON, evidence index, dispatch event, or checkpoint ledger row is rewritten.
- Subsequent reports must name the exact command before presenting a test count. Aggregating counts across commands is prohibited unless the aggregate tool itself produced that total.
