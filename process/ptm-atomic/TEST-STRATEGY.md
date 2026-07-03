---
project_id: "atomic-ops-ngfw-install-config-chain"
wave_scope: "W1-W2 / STORY-001..005 / CR-003"
created_at: "2026-05-18T18:15:56+0800"
owner: "meta-qa"
batch_id: "CP7-STORY-001-005"
---

# Test Strategy

## Scope

This strategy covers CP7 verification for:

| Story | Verification focus |
|---|---|
| STORY-001 | Schema v1.1 contract, schema field docs, error codes, naming docs, and baseline atom compatibility. |
| STORY-002 | Five NGFW install/init/login/health atoms, `ngfw_installation` package references, high-risk gates, and sensitive-data boundaries. |
| STORY-003 | Ten capacity domains, twenty config/verify op_ids, `ngfw_capacity_config` and `ngfw_verification` package references, and D-004 package scope. |
| STORY-004 | Ten multi-device batch config atoms, `ngfw_batch_config` package references, batch contract, concurrency, idempotency, failure isolation, and high-risk gates. |
| STORY-005 | Read-only security gate script, fixture exit-code coverage, sensitive-output redaction, and CLI command boundary. |

STORY-006 is explicitly out of scope and remains blocked until all five upstream CP7 results exist.

## Entry Gate

| Entry criterion | Status | Evidence | Impact |
|---|---|---|---|
| Story status is `ready-for-verification` | PASS | `process/STORY-STATUS.md` lists STORY-001..005 as ready-for-verification. | Verification can inspect implemented artifacts. |
| CP6 coding checks are complete | PASS | `process/checks/CP6-STORY-001..005-*-CODING-DONE.md` all have `status: "PASS"`. | Implementation handoff and coding evidence exist. |
| CP5 LLD batch is approved | PASS | `checkpoints/CP5-ALL-STORIES-LLD-BATCH.md` has `status: "approved"`. | LLDs are valid verification inputs. |
| Validation environment is approved | BLOCKED | `process/VALIDATION-ENV.yaml` is missing. | CP7 can run read-only checks, but PASS release should be blocked until the validation environment is supplied and approved. |
| Allowed write set is constrained | PASS | User instruction allows only this strategy, the batch report, and five CP7 files. | Product files must remain untouched. |

## Test Design Method Selection

| Method | Applicability | Application in this batch |
|---|---|---|
| Equivalence partitioning | High | Partition checks by atom category: install/init/login/health, single-device capacity config, single-device verification, multi-device batch config, packages, docs, scripts, and CLI. |
| Boundary value analysis | High | Check schema version `1.1`, batch `max_concurrency` default 1 and maximum 5, exit codes 0/31/32/33, and expected counts: 5 install atoms, 10 capacity domains, 20 config/verify atoms, 10 batch atoms. |
| State transition testing | Medium | Validate contract-only state progression: install -> init -> login -> login-state check -> health verification; config -> verification; batch partial failure -> diagnostics/manual handling. |
| Error guessing | High | Probe likely defects: missing package references, unsafe real-device CLI verbs, sensitive literals, missing high-risk gates, invalid YAML/schema, docs drifting from schema, and malformed security-gate inputs. |

## ISO 25010 Priorities

| Quality characteristic | Priority | Verification focus | CP7 dimension |
|---|---|---|---|
| Functional suitability | P0 | Required atoms, packages, schema fields, docs, and scripts exist and match Story acceptance criteria. | Completeness and acceptance coverage. |
| Reliability | P0 | Repository-level schema/layout/security checks and CLI help complete without error. | Static validation and integration. |
| Security | P0 | No sensitive literals, no high-risk atom without gate, no real-device CLI command, no product copy from `.input/`. | Security compliance. |
| Maintainability | P1 | Naming conventions, package references, and docs stay aligned with schema and atom op_ids. | Naming and reference integrity. |
| Portability | P2 | Not an Agent/Skill install package; validation uses README-native repo surfaces and `uv`. | Platform boundary. |
| Usability | P2 | Docs explain fields, error codes, naming, and batch contract sufficiently for STORY-006 documentation follow-up. | Documentation coverage. |
| Compatibility | P2 | Existing CLI command family remains `sync/list/show/packages/validate`; no executor verbs introduced. | CLI boundary. |
| Performance efficiency | P3 | Batch contracts keep max concurrency bounded; static checks stay local/read-only. | Batch and script boundary. |

## Quality Gates

### Entry Criteria

| Criterion | Required outcome |
|---|---|
| STORY-001..005 status | `ready-for-verification` |
| CP6 per Story | `PASS` with Agent Dispatch Evidence |
| CP5 LLD | Approved and `confirmed=true` for each target LLD |
| Validation environment | `process/VALIDATION-ENV.yaml` exists and `approval.confirmed=true` |
| Write boundary | Only mandatory CP7 outputs are modified |

### Exit Criteria

| Criterion | Required outcome |
|---|---|
| Repository static checks | `validate_schema.py`, `layout_check.py`, `security_gate_check.py`, and `atomic-ops --help` pass. |
| Story acceptance checks | Every acceptance criterion has explicit evidence or a documented blocker. |
| Security checks | Sensitive findings = 0 and high-risk gate findings = 0 for the default security scan. |
| CP7 evidence | Each CP7 file includes Entry Criteria, Checklist, Exit Criteria, Deliverables, Agent Dispatch Evidence, Commands run and key outputs, Defects / risks, and Conclusion. |
| STORY-006 gate | Remains blocked until STORY-001..005 CP7 outcomes are written and accepted by meta-po. |

Because `process/VALIDATION-ENV.yaml` is absent at strategy creation time, any CP7 outcome that otherwise passes functional verification must still record a validation-environment blocker unless meta-po supplies an approved environment before CP7 finalization.

## Command Plan

| Command | Purpose |
|---|---|
| `uv run --python 3.11 python scripts/validate_schema.py atoms` | Validate all atom YAML files against the schema. |
| `uv run --python 3.11 python scripts/layout_check.py` | Validate repository layout, atom/package naming, and reference structure. |
| `uv run --python 3.11 python scripts/security_gate_check.py` | Validate sensitive-data and high-risk gate rules. |
| `uv run atomic-ops --help` | Confirm CLI remains available and read-only command boundary is inspectable. |
| Additional read-only `uv run --python 3.11 python -c ...` probes | Count and cross-check package op_ids, schema/docs alignment, exit-code fixtures, and forbidden CLI verbs. |
