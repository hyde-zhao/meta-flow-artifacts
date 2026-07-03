---
story_id: "CR037-S07"
cr_ref: "CR-037"
stage: "CP6"
status: "implemented"
implementation_actor: "meta-dev"
artifact_actor: "host-orchestrator"
dispatch_ref: "D-CR037-META-DEV-CP6-S07"
inline_fallback: false
created_at: "2026-07-03T17:27:30+08:00"
---

# CR037-S07 CP6 Implementation Summary

## Scope

CR037-S07 implements the feature/capability registry and resolver contract for CR-037 project governance. The implementation extends the existing `meta_flow/design/feature_registry.py` module rather than creating a second registry namespace.

The implementation adds:

- `CAPABILITY-REGISTRY.yaml` path support and capability registry validation.
- Feature registry v1/v2 compatibility for canonical `id`, legacy `feature_id`, aliases, `deprecated_by`, and `source_refs`.
- Registry finding and resolver result models with stable `code`, `severity`, `input_ref`, and `canonical_id` fields.
- `resolve_ref()`, `resolve_refs()`, and `build_candidate_report()` APIs.
- `meta-flow feature check --include-capabilities` and `meta-flow feature resolve` CLI behavior through the existing feature command path.
- Tests covering resolver states, alias conflict, deprecated refs, unresolved refs, sensitive-value rejection, candidate reports, and synthetic downstream consumers.

## Changed Files

| Path | Role |
|---|---|
| `meta_flow/design/feature_registry.py` | Feature/capability registry validation, resolver API, candidate report, and CLI subcommands. |
| `tests/test_feature_registry.py` | Registry v2, capability registry, resolver, candidate report, downstream consumer, and CLI contract tests. |

`meta_flow/cli.py` already forwarded the top-level `feature` command to `feature_registry.main()` and did not require an S07 code change.

## Design Contract Mapping

| Contract | Implementation | Verification |
|---|---|---|
| Feature registry remains backward compatible. | `validate_registry()` accepts schema version 1 or 2 and resolves both `id` and `feature_id`. | `test_feature_registry_v2_supports_id_aliases_and_source_refs`. |
| Capability refs must resolve through a standard registry. | `CAPABILITY_REGISTRY_REL`, `load_registry(..., kind="capability")`, and `validate_registry(..., include_capabilities=True)`. | `test_capability_registry_check_validates_refs_and_sensitive_values`. |
| Resolver distinguishes resolved, unresolved, deprecated, and conflict. | `ResolverResult`, `resolve_ref()`, and `resolve_refs()`. | `test_resolver_returns_unresolved_deprecated_and_conflict_results`. |
| Unknown refs must not auto-create canonical IDs. | `build_candidate_report()` returns candidate-only entries and does not write registry files. | `test_candidate_report_does_not_create_canonical_capability_id`. |
| Downstream consumers must consume resolver results. | Synthetic impact and roadmap consumers normalize only resolved refs and block unresolved refs. | `test_synthetic_downstream_consumers_must_use_resolver_results`. |
| Fixtures must use governance placeholder IDs. | New capability fixture uses `CAP-PG-REGISTRY-REFS`; no quant-lab real ID is introduced. | Test fixtures and S07 diff review. |
| Dev agent must not write CP6 process artifacts. | Dev agent modified source/tests only; Host Orchestrator generated CP6 process evidence after validation. | Dispatch final result and pre-artifact process diff check. |

## Validation

Commands run by host-orchestrator after the child agent completed source and test changes:

```bash
PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_feature_registry.py
PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow feature --help
PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow state check --project-root . --mode enforce
git diff --check -- meta_flow/design/feature_registry.py meta_flow/cli.py tests/test_feature_registry.py
git diff -- process/returns process/evidence process/checks process/state process/handoffs
git diff -- process/quant-lab docs
```

Results:

- Target tests: `15 passed in 0.05s`.
- `meta-flow feature --help`: feature check and resolve commands are visible.
- `meta-flow state check --mode enforce`: `State v2 Check: OK`.
- `git diff --check`: no whitespace errors.
- Pre-artifact process diff for return/evidence/checks/state/handoffs: no output.
- `process/quant-lab` and `docs` diff: no output.

## Process Artifact Ownership

The S07 handoff intentionally told the dev agent not to write `process/returns/**`, `process/evidence/**`, `process/checks/**`, or `process/state/**`. This Story follows the S07+ execution strategy: the child dev agent owns code and tests; Host Orchestrator runs validation and writes CP6 process artifacts.

## Remaining Risks

| Risk | Level | Status | Mitigation |
|---|---|---|---|
| `R-CR037-SECOND-MECHANISM` | MEDIUM | carried | S07 extends the existing `feature_registry.py` path; it does not introduce a separate capability namespace outside registry governance. |
| `R-CR037-REGISTRY-DRIFT` | MEDIUM | reduced | Resolver and checker provide stable result codes and block unresolved/free-string refs; downstream S08/S09/S10/S12/S13 must consume these APIs. |
| `R-CR037-YAML-SUBSET` | LOW | carried | Registry parser continues to use JSON-compatible YAML, matching existing feature registry behavior. |
| `R-CR037-QA-SUBAGENT-NO-OUTPUT` | MEDIUM | carried | CP6 process artifacts are Host-owned by design for S07+, avoiding repeated child-agent failures on process artifact writing. |

## Next Route

Host-orchestrator should validate the CP6 return/evidence/result, append the CP6 result to the checkpoint ledger, mark S07 `ready-for-verification`, then prepare S07 CP7 verification under the established host-owned process artifact strategy.
