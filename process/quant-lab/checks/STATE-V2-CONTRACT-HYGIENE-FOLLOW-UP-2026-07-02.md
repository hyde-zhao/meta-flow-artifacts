---
check_id: "STATE-V2-CONTRACT-HYGIENE-FOLLOW-UP-2026-07-02"
status: "open"
severity: "medium"
created_at: "2026-07-02T07:45:00+08:00"
owner: "host-orchestrator"
related_cr: "CR-151"
follow_up_type: "state_hygiene"
---

# STATE v2 Contract Hygiene Follow-up

## Finding

`meta-flow state check --project-root /home/hyde/workspace/quant-lab` currently fails with v2 contract errors:

```text
State v2 Check: FAIL
- ERROR: STATE.current.json too large: 115.2 KB > 20.0 KB
- ERROR: STATE.current.json must not store long-running field: checkpoints
- ERROR: STATE.current.json must not store long-running field: human_gate_decisions
- ERROR: STATE.md too large: 25.8 KB > 12.0 KB
```

## Assessment

This is not caused by CR151 source implementation, but it is a real STATE v2 contract violation, not a warning. It should not be described as merely non-blocking hygiene.

CR151 CP6 / CP7 can remain valid because their machine truth sources are the CP result JSON files, Story return packets, evidence indexes and ledgers. However, CP8 release readiness must explicitly account for this risk if it consumes `STATE.current.json` as a machine state entrypoint.

## Routing

| Field | Value |
|---|---|
| Recommended route | Create or attach to a STATE v2 slimming / retention follow-up before or during CP8 readiness. |
| Immediate CR151 source-code impact | None. |
| CP8 impact | Must be listed as process evidence risk / readiness caveat until resolved. |
| Suggested remediation | Move long-running `checkpoints` and `human_gate_decisions` payloads to ledgers or compact refs; keep `STATE.current.json` under the v2 size budget. |

## Status

Open. Not fixed in CR151 CP6/CP7 because it is a cross-workflow state-contract cleanup, not a local/static/fixture strategy framework source change.
