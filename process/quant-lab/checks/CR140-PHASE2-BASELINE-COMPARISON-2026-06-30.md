# CR140 Phase 2 Baseline Comparison

## Scope

Phase 2 helper consolidation 提交后，在 source repo 与 artifact repo 均 clean 的状态下复跑全量 pytest，对比 Phase 0 failure set。

## Precondition

- source repo `git status --short`：clean。
- artifact repo `git status --short`：clean。
- 未访问真实数据湖 / NAS / provider / catalog pointer / QMT / live / runtime。

## Commands

| 命令 | 结果 | 说明 |
|---|---:|---|
| `uv run --python 3.11 pytest -q --tb=no -p no:cacheprovider` | `46 failed, 1428 passed` | 回到 Phase 0 基线计数。 |

## Failure Set Comparison

新增失败：无。

修复的基线失败：无。

保持失败：46 条，和 `process/checks/CR140-PHASE0-BASELINE-FAILURES.txt` 一致。

## Decision

Phase 2 可以关闭并进入 Phase 3。Phase 3 只能使用合成 fixture 或注入层证明 turnover 新旧实现重构等价；不得声明真实数据研究语义已验证，不得访问真实 lake。
