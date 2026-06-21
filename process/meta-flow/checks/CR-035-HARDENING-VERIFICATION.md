# CR-035 Hardening Verification

## Scope

- Fix `workspace bootstrap --artifact-root <relative>` so relative artifact roots are resolved from `--project-root`.
- Fix `cr bootstrap --readiness` propagation to CR frontmatter, JSON index, legacy YAML index, summary, and CR ledger.
- Confirm pytest environment remains healthy after CR-034 dependency cleanup.

## Commands

| Command | Result |
|---|---|
| `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 pytest -q tests/test_workspace_routing.py tests/test_cr_lifecycle.py` | PASS, 10 tests |
| `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 pytest -q` | PASS, 126 tests |
| `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 python scripts/check_delivery_guardrails.py` | PASS |
| `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow state check --project-root .` | PASS |
| `PYTHONDONTWRITEBYTECODE=1 uv run --python 3.11 meta-flow check cr-tracking --project-root .` | PASS |
| `git diff --check` | PASS |
| Temporary target smoke: `workspace bootstrap --project-root /tmp/.../target-project --artifact-root ../artifacts --project-name target-project` | PASS; artifact root resolved to `/tmp/.../artifacts` |
| Temporary target smoke: `cr bootstrap --readiness ready_with_risk` | PASS; readiness appears in CR, JSON index, YAML index, and CR ledger |

## Decision

CR-035 can close as `READY`. CR-033 remains deferred and is not started.
