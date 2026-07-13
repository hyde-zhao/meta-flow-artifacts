# CR-045 R6 Fixes

- Replaced the weaker approved-CP8 delivered check with the shared complete-terminal predicate.
- Added negative coverage for wrong stop reason, residual pending gate, active CR, wrong phase, and interrupted states that retain a gate.
- Preserved valid authorization and workflow-health interruption routes when no gate is pending.
- Replaced the active user-manual `cp8_approved` example with `cp8_closed`.
- Preserved historical append-only ledger entries containing the old value; the latest canonical status-sync event is `cp8_closed`.
