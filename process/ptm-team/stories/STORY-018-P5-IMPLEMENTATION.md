# STORY-018-P5 Implementation Evidence

## Scope

- CR-018 follow-up for ptm-tde Gate contract hardening.
- Implements user feedback:
  - Every normal and abnormal scenario-chain step must map to an atomic operation.
  - Newly proposed atomic operations must be explicitly shown for Gate confirmation.
  - Candidate test factors and candidate atomic operations must be explicitly shown and confirmed by the user.

## Changes

- `skills/checkpoint-manager/scripts/run_checkpoint.py`
  - GATE-2 validates step-level atomic operation refs inside `normal_path` and `abnormal_path`.
  - Step refs must use `action_source_ref(s)`, `atomic_op`, or `op_id`, and must link back to scenario-level `action_source_refs` when available.
  - GATE-2 renders a new atomic operation candidate confirmation section when candidate/new atomic markers are present.
  - GATE-3 blocks when candidate factor or atomic-op sources exist but `mfq/candidates/` summaries do not record user confirmation decisions.
- `tests/test_cr018_p2.py`
  - Added regression coverage for missing step atomic refs, GATE-2 new atomic display, and GATE-3 candidate confirmation.
- Updated ptm-tde docs, agent prompt, checkpoint-manager skill, scenario-discovery skill, test-point-integrator skill, and runtime artifact contract.

## Verification

- `python3 -m unittest tests.test_cr018_p2.CR018P2Tests.test_gate2_passes_with_normal_and_abnormal_paths tests.test_cr018_p2.CR018P2Tests.test_gate2_blocks_when_normal_step_lacks_atomic_ref tests.test_cr018_p2.CR018P2Tests.test_gate2_displays_new_atomic_candidates_for_manual_confirmation tests.test_cr018_p2.CR018P2Tests.test_gate3_blocks_when_candidates_lack_user_confirmation tests.test_cr018_p2.CR018P2Tests.test_gate3_passes_when_candidates_have_user_confirmation`
- `python3 -m compileall skills/checkpoint-manager/scripts/run_checkpoint.py tests/test_cr018_p2.py`
- `python3 -m unittest discover -s tests -p 'test*.py'` -> 13 tests passed
- `git diff --check`

## Remaining Risk

- The Gate checker remains a Markdown/YAML marker-based machine baseline. It intentionally catches missing contracts and confirmation status without trying to fully parse arbitrary human prose.
