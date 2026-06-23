# STORY-018 P3 Implementation Evidence

## Scope

CR-018 P3 整改覆盖 ptm-tde PC 结构化步骤契约与 GATE-4 强校验：修复最终测试用例列混乱、测试步骤未映射原子操作仍可放行的问题。

## Implementation Objects

| Object | Files | Change |
|---|---|---|
| Checkpoint guardrail | `skills/checkpoint-manager/scripts/run_checkpoint.py` | GATE-4 从“任意 Markdown 行 >=16 列”升级为标准 16 列表头、所有数据行恰好 16 列、`case_steps`、`atomic_op.op_id` 与 `action_source_refs` 回链检查 |
| Tests | `tests/test_cr018_p2.py` | 新增 GATE-4 通过夹具、缺 `atomic_op` 阻断、未转义 `|` 导致列错位阻断 |
| Agent / docs / skill contract | `agents/ptm-tde.md`, `docs/ptm-tde/README.md`, `docs/ptm-tde/USER-MANUAL.md`, `docs/ptm-tde/gate-spec.md`, `skills/checkpoint-manager/SKILL.md`, `skills/deliverable-renderer/SKILL.md` | 统一 PC `case_steps` + 16 列交付契约，移除“PC→原子操作隐式/后续”口径 |

## Contract Mapping

| Contract | Implementation |
|---|---|
| PC 源契约必须是结构化步骤 | `validate_pc_step_contract()` 校验 `case_steps / step_name / atomic_op.op_id / action_source_refs` |
| Markdown 16 列必须稳定 | `validate_standard_pc_tables()` 校验标准 16 列表头、每行恰好 16 列、步骤列包含 `原子操作：` |
| 最终交付不得再次错列 | GATE-4 对 `*测试用例*.md` 要求唯一标准 16 列汇总表 |
| renderer 不应拼接旧 Markdown 行 | `deliverable-renderer` 契约要求从结构化 PC 字段重新生成 16 列行 |

## Verification Results

| Check | Result |
|---|---|
| `uv run python -m unittest discover -s tests -p 'test*.py'` | PASS，6 tests |
| `uv run python -m compileall skills/checkpoint-manager/scripts/run_checkpoint.py tests/test_cr018_p2.py` | PASS |
| `python3 -m unittest discover -s tests -p 'test*.py'` | PASS，6 tests |
| `python3 -m compileall skills/checkpoint-manager/scripts/run_checkpoint.py tests/test_cr018_p2.py` | PASS |
| `git diff --check` | PASS |
| Residual old-contract grep | PASS，未发现旧 `>=16 列` / `隐式引用 atomic-ops` / `PC → 原子操作 已有（隐式）` 口径 |

## Remaining Risk

`docs/ptm-tde/data-flow-spec.md` 已在工作区同步更新，但该文件被 `docs/.gitignore` 忽略，不属于当前产品 diff。若团队希望把 data-flow spec 纳入正式交付，需要单独调整 docs 跟踪策略。
