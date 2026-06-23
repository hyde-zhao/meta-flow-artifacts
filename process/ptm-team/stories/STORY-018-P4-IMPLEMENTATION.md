# STORY-018-P4 Implementation Evidence

## Scope

- CR-018 follow-up for ptm-tde GATE-2.
- Fixes confirmed-scenarios deliverables passing with only file-level markers while individual scenarios lack normal and abnormal chains.

## Changes

- Added per-scenario confirmed scenario contract validation in `skills/checkpoint-manager/scripts/run_checkpoint.py`.
- GATE-2 now splits `confirmed-scenarios.md` by `scenario_id` or scenario heading and checks each block for:
  - `normal_path` with `step_id / sub_step_ids / operation / necessity / description`
  - `necessity` value in `必要 / 可选 / 至少选择一项`
  - `abnormal_path` with `abnormal_item / related_normal_steps / input_or_state / expected_handling`, or explicit N/A reason
  - `minimal_logic_chain`
  - atomic operation and `action_source_refs`
- Added regression tests for valid scenario chains, missing normal/abnormal chains, and missing `related_normal_steps`.
- Updated GATE-2 documentation and ptm-tde execution guidance to state the per-scenario blocking rule.

## Verification

- `python3 -m unittest tests.test_cr018_p2.CR018P2Tests.test_gate2_passes_with_normal_and_abnormal_paths tests.test_cr018_p2.CR018P2Tests.test_gate2_blocks_when_scenario_lacks_normal_and_abnormal_paths tests.test_cr018_p2.CR018P2Tests.test_gate2_blocks_when_abnormal_path_lacks_related_steps`
- `python3 -m compileall skills/checkpoint-manager/scripts/run_checkpoint.py tests/test_cr018_p2.py`
- `python3 -m unittest discover -s tests -p 'test*.py'` -> 9 tests passed
- `git diff --check`

## Remaining Risk

- The validator is markdown/YAML marker based, not a full schema parser. It intentionally blocks obvious missing chains while preserving flexible generated Markdown formats.
