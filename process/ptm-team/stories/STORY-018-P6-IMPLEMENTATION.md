# STORY-018-P6 Implementation Evidence

## Scope

- Follow-up fix from the BGP4+ ptm-tde evaluation run.
- Hardens ptm-tde against these observed failures:
  - MFQ stage artifacts were internally inconsistent but still easy to advance.
  - F analysis could be interrupted or partial while later artifacts were manually completed without a clear guardrail.
  - Candidate summaries with `decision=pending-review` could satisfy the previous GATE-3 marker check because it only looked for a `decision` marker.
  - GATE-3 PASS left `process/STATE.yaml.current_step` at `test-point-integrator`.
  - CP03-CP07 only checked directories, not stage output contracts.

## Changes

- `skills/checkpoint-manager/scripts/run_checkpoint.py`
  - Adds `mfq/candidates/` to required initialized directories.
  - Changes candidate confirmation checks so `pending-review`, `pending`, `TBD`, `todo`, and Chinese pending states do not pass GATE-3.
  - Requires final candidate decisions to be `confirmed / rejected / modified` or clear Chinese equivalents.
  - Sets `current_step: "mfq-exit"` after GATE-3 machine-baseline PASS.
  - Adds field-level CP03-CP07 checks for CAE, trace, `fact_status`, LC bindings, tool-analysis, design-plan, and candidate summaries.
- `agents/ptm-tde.md`
  - Adds small-slice MFQ execution protocol.
  - Requires CP03-CP07 after each MFQ skill.
  - Forbids presenting partial or interrupted skill output as completed skill execution.
  - Requires candidates to remain `decision=pending-review` before explicit user confirmation.
- `skills/test-point-integrator/SKILL.md`
  - Requires pre-confirmation candidate summaries with `decision=pending-review` and `decision_basis=awaiting-user-review`.
  - Clarifies that pending summaries are valid artifacts but cannot pass GATE-3.
  - Removes the workflow conflict where missing `mfq/candidates/` caused evidence loss; the directory is now initialized by checkpoint-manager/GATE-1.
- `skills/m-analyzer/SKILL.md`
  - Adds `fact_status` and statistics consistency rules for `confirmed / needs-confirmation / not-generated`.
- `skills/f-analyzer/SKILL.md`
  - Requires coupling graph/test-point `fact_status` consistency and forbids upgrading inferred/gap-based coupling to `confirmed`.
- `skills/q-analyzer/SKILL.md`
  - Requires quality TP/tool-analysis uncertainty consistency and forbids upgrading OPEN/candidate/tool-gap quality points to `confirmed`.
- `skills/design-planner/SKILL.md`
  - Requires `M1-only / not-generated / needs-confirmation` scope limits to remain visible in design-plan and reasoning.
- `docs/ptm-tde/gate-spec.md`, `docs/ptm-tde/runtime-artifacts.md`, `docs/ptm-tde/USER-MANUAL.md`
  - Documents the pending-review vs final-decision split.
- `tests/test_cr018_p2.py`
  - Adds regression coverage for `pending-review` candidate summaries blocking GATE-3.
  - Updates GATE-3 PASS fixture to include `fact_status` and assert `current_step: "mfq-exit"`.

## Verification

| Command | Result |
|---|---|
| `uv run python -m unittest tests.test_cr018_p2` | PASS, 19 tests |
| `uv run python -m unittest discover -s tests` | PASS, 23 tests |
| `uv run python -m py_compile skills/checkpoint-manager/scripts/run_checkpoint.py` | PASS |
| `git diff --check` | PASS |
| `uv run python /home/hyde/projects/ptm-team/script/install.py --source-dir /home/hyde/projects/ptm-team check claude --agent ptm-tde` from `/home/hyde/projects/ptm-tde/test` before reinstall | SOURCE_DRIFT detected for expected changed assets |
| `uv run python /home/hyde/projects/ptm-team/script/install.py --source-dir /home/hyde/projects/ptm-team install claude --agent ptm-tde` from `/home/hyde/projects/ptm-tde/test` | PASS |
| `uv run python /home/hyde/projects/ptm-team/script/install.py --source-dir /home/hyde/projects/ptm-team check claude --agent ptm-tde` from `/home/hyde/projects/ptm-tde/test` after reinstall | PASS, no drift |

## Remaining Risk

- GATE-3 remains a marker/field-level machine baseline. It now blocks the concrete failures found in the BGP4+ run, but semantic correctness of MFQ reasoning, LC quality, and candidate decisions still requires the manual GATE-3 review.
- Existing generated BGP4+ artifacts under `/home/hyde/projects/ptm-tde/test/bgp4+` were not regenerated after this source fix. Future runs will use the updated Claude installation.
