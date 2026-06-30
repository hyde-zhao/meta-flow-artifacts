# CR140 Phase 0 Pytest Baseline

## Entry Criteria

- `process/context/EXPERIMENTS-REFACTOR-PLAN-2026-06-28.md` 已读取并作为范围基线。
- `git status --short` 在项目源码 worktree 中为空。
- `uv run --python 3.11 meta-flow workspace check` 通过。
- 已登记 `CR-140`，并明确不授权真实 lake/NAS/provider/catalog/QMT/live/runtime。

## Checklist

| 项 | 结果 | 证据 |
|---|---|---|
| workspace route health | PASS | `process_link_health: ok` |
| source worktree dirty check | PASS | `git status --short` 输出为空 |
| artifact dirty boundary | PASS_WITH_EXPECTED_CHANGES | 仅 CR140 登记与 Phase 0 基线文件变更 |
| full pytest baseline | BASELINE_RED | `46 failed, 1428 passed in 49.07s` |
| failure set recorded | PASS | `process/checks/CR140-PHASE0-BASELINE-FAILURES.txt` |

## Command

```bash
uv run --python 3.11 pytest -q --tb=no -p no:cacheprovider
```

## Result

- Exit code: `1`
- Summary: `46 failed, 1428 passed in 49.07s`
- Failure set: `process/checks/CR140-PHASE0-BASELINE-FAILURES.txt`

## Exit Criteria

- Phase 0 baseline is recorded.
- Later Phase verification must compare against this failure set.
- Passing condition for later Phases: failure set is unchanged or smaller; any new failing node id is blocking.

## Deliverables

- `process/checks/CR140-PHASE0-PYTEST-BASELINE-2026-06-30.md`
- `process/checks/CR140-PHASE0-BASELINE-FAILURES.txt`

## Authorization Boundary

This run did not intentionally link a real data lake, mount NAS, fetch provider data, write lake objects, write catalog pointers, or run QMT/live/runtime operations. If a future validation requires real lake readonly access, it must be separately authorized before execution.
