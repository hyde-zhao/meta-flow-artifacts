# CP6 CR-008 Rollback Coding Done

## Entry Criteria

- CR-008 范围已记录。
- HLD 与需求摘要已生成。
- 目标文件范围清晰。

## Checklist

| 项 | 结果 | 证据 |
|---|---|---|
| schema 字段实现 | PASS | `schemas/atomic-op.schema.yaml` |
| rollback 规则模块实现 | PASS | `src/atomic_ops/rollback.py` |
| validate / rollback CLI 实现 | PASS | `src/atomic_ops/commands/validate.py`, `src/atomic_ops/commands/rollback.py` |
| list/show 元数据暴露 | PASS | `src/atomic_ops/repository.py`, `src/atomic_ops/commands/list_ops.py` |
| run envelope rollback_context | PASS | `src/atomic_ops/commands/run.py` |
| 首批 atom 契约 | PASS | policy-route/interface/tg 10 个 atom |
| 单测补充 | PASS | `tests/test_runner.py` |

## Exit Criteria

CP6 PASS。实现已完成，进入验证。

## Deliverables

- `process/stories/STORY-CR-008-ROLLBACK-IMPLEMENTATION.md`
- 代码、schema、atom 和文档 diff。
