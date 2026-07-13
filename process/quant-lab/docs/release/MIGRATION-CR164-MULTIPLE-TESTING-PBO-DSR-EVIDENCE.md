# CR-164 Migration

N/A. CR-164 introduces additive in-process Python contracts and calculators only. It changes no database, persisted production schema, dependency, environment variable, platform path, installer or historical evidence. Existing callers remain fail-closed until they explicitly supply a locally validated `StatisticalEvidenceSummary`.
