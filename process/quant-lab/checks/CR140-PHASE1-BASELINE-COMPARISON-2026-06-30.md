# CR140 Phase 1 Baseline Comparison

## Scope

Phase 1 完成后，在 source repo 与 artifact repo 均 clean 的状态下复跑全量 pytest，对比 Phase 0 failure set。

## Precondition

- source repo `git status --short`：clean。
- artifact repo `git status --short`：clean。
- CR132 process artifact hygiene：`17 passed`，`unclassified=0`。
- 未访问真实数据湖 / NAS / provider / catalog pointer / QMT / live / runtime。

## Commands

| 命令 | 结果 | 说明 |
|---|---:|---|
| `uv run --python 3.11 pytest -q -p no:cacheprovider tests/test_cr132_process_artifact_hygiene.py` | `17 passed` | dirty-workspace hygiene 已恢复。 |
| `uv run --python 3.11 python scripts/check_process_artifact_hygiene.py --source-root . --process-root process --json` | PASS | `passed=true`，`unclassified=0`。 |
| `uv run --python 3.11 pytest -q --tb=no -p no:cacheprovider` | `46 failed, 1428 passed` | 回到 Phase 0 基线计数。 |

## Failure Set Comparison

新增失败：无。

修复的基线失败：无。

保持失败：46 条，和 `process/checks/CR140-PHASE0-BASELINE-FAILURES.txt` 一致。

## Notes

迁移中途在未提交状态下曾出现 `48 failed, 1426 passed`，新增的 2 个失败为 CR132 dirty-workspace hygiene 对未提交 source / artifact 变更的预期拦截。source Phase 1 切片提交与 artifact evidence 提交后，该临时失败已清零。

## Decision

Phase 1 可以关闭并进入 Phase 2。Phase 2 必须先执行 helper 等价 diff，再删除本地 helper；非等价 helper 不删除，改为登记保留原因。
