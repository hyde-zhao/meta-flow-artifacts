# CR-046 CP2 R2 Host Observations

## OBS-R2-001 Checkpoint attempt event identity

- Status: `accepted-as-existing-story-fixture`
- Severity: HIGH
- Observation: `meta-flow cp ledger-append` emitted `event_id=CP1-CR-046` and `CP2-CR-046` for both R1 and R2, so `meta-flow event check` rejected the valid second attempts as duplicates.
- Disposition: This does not add a new product Story or requirement. It is a concrete dogfooding fixture for existing Story 3 / CP result attempt-final correlation. Each check attempt needs a unique event identity while preserving checkpoint identity, result supersession and attempt correlation.
- Immediate evidence correction: only the newly appended R2 events were assigned `CP1-CR046-R2` / `CP2-CR046-R2` and annotated with `check_attempt=2` plus `supersedes_result_ref`; R1 events were not modified.
