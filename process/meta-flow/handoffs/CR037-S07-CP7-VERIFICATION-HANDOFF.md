---
handoff_id: H-CR037-S07-CP7-VERIFICATION
cr_id: CR-037
story_id: CR037-S07
from_role: host-orchestrator
to_role: meta-qa
stage: story-verification
status: completed
created_at: 2026-07-03T17:40:18+08:00
completed_at: 2026-07-03T17:40:18+08:00
context_ref: process/context/stories/CR037-S07.CP7.verify-packet.json
dispatch_id: D-CR037-META-QA-CP7-S07
canonical_role: meta-qa
codex_agent_name: host-orchestrator-inline-fallback
capsule_first: true
read_profile: compact
inline_fallback: true
---

# CR037-S07 CP7 Verification Handoff

## Objective

Verify CR037-S07 after CP6 PASS. The verifier must prove that registry-backed feature/capability refs, capability registry validation, resolver behavior, candidate report behavior, and synthetic downstream consumer contracts satisfy the approved Story contract.

## Required Context

Read the capsule first:

- `process/context/stories/CR037-S07.CP7.verify-packet.json`

Read full LLD or implementation files only if the capsule, CP6 return packet, or evidence index is insufficient, and record any full-document expansion in `process/state/READ-EXPANSION-LEDGER.ndjson`.

## Verification Focus

- `FEATURE-REGISTRY.yaml` v1/v2 compatibility remains backward compatible.
- `CAPABILITY-REGISTRY.yaml` checker validates canonical IDs, aliases, `deprecated_by`, `feature_refs`, owner/source refs, and credential-like content.
- Resolver returns stable resolved / unresolved / deprecated / conflict results.
- Unknown refs become blocked findings or candidate report entries; resolver never auto-creates canonical IDs.
- Synthetic downstream consumers use resolver output instead of free strings.
- Fixture IDs use `CAP-PG-*` and do not introduce quant-lab real capability IDs.
- `process/quant-lab/**`, `docs/**`, runtime, publish, live, production write, and credential reads remain forbidden.

## Allowed Writes

- `process/returns/CR037-S07.CP7.return.json`
- `process/evidence/CR037-S07.CP7.index.json`
- `process/checks/CP7-CR037-S07-VERIFICATION-DONE.result.json`
- `process/checks/CP7-CR037-S07-VERIFICATION-DONE.result.summary.md`

## Forbidden Writes

- `meta_flow/**`
- `tests/**`
- `process/quant-lab/**`
- `docs/**`
- `delivery/**`
- `.env`
- any credential, secret, token, cookie, or private-key path

## Required Verification Commands

```bash
PYTHONDONTWRITEBYTECODE=1 PYTEST_ADDOPTS='-p no:cacheprovider' uv run --python 3.11 pytest -q tests/test_feature_registry.py
PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow feature --help
PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow state check --project-root . --mode enforce
PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow cp result-check --result process/checks/CP6-CR037-S07-CODING-DONE.result.json --project-root .
PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow story return-check --packet process/context/stories/CR037-S07.CP6.work-packet.json --return process/returns/CR037-S07.CP6.return.json --project-root .
PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow story evidence-check --index process/evidence/CR037-S07.CP6.index.json --project-root .
git diff --check -- meta_flow/design/feature_registry.py meta_flow/cli.py tests/test_feature_registry.py
git diff -- process/quant-lab docs
```

## Completion Note

CP7 was completed by user-authorized host-orchestrator inline-fallback. This follows the approved S07+ execution strategy: child dev agents write source/tests, while Host Orchestrator owns process evidence generation and verification artifacts.
