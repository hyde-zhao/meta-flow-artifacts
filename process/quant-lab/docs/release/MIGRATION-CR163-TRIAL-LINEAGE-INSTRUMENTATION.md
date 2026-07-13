---
change_id: "CR-163"
migration_required: false
profile: "compact"
created_at: "2026-07-11T18:15:00+08:00"
updated_at: "2026-07-11T18:38:00+08:00"
---

# CR163 Migration and Compatibility

No state or production-data migration is required.

| Surface | Decision |
|---|---|
| Historical CR155 evidence | no migration, inference, reconstruction or backfill |
| Existing uninstrumented runs | remain `typed_unavailable` / blocked |
| New lineage JSON/JSONL | created only by future explicitly instrumented local runs |
| State schema / database | no migration |
| Environment / credentials | no new configuration and no access authorized |
| Install paths / dependencies | unchanged |
| CLI arguments | optional instrumentation inputs only; absence fails closed |
| Public Python contracts | additive family/manifest/validation/projection objects; positive truth requires binding |
| Admission compatibility | manual counts cannot create present truth; effective/C1 remains unavailable |

If later work enables effective-trial computation, use existing `FU-CR161-002`; do not create a duplicate CR163 candidate. Historical backfill, real runtime entrypoints or external persistence require their own approved CR and migration plan.
