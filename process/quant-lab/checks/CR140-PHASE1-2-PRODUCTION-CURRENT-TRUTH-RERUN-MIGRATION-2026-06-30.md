# CR140 Phase 1.2 Production Current Truth Rerun Migration

## Scope

- 迁移 `experiments/production_current_truth_rerun.py` 到 `engine/production_current_truth_rerun.py`。
- 更新 `tests/test_cr018_production_current_truth_rerun.py` 导入路径。

## Responsibility Decision

该文件不是实验入口，也没有 CLI；它是 CR018-S08 production current truth research rerun 的 dry-run contract，复用 `engine.research_dataset` 组装 release-bound fixture / payload。按职责归入 `engine/`，而不是 `scripts/data_lake/`。

## Boundary

- 未访问真实数据湖。
- 未访问 NAS。
- 未触发 provider fetch。
- 未写 lake / catalog pointer。
- 未触发 QMT / MiniQMT / xtquant / gateway / simulation / live / trading runtime。

## Commands

| 命令 | 结果 | 说明 |
|---|---:|---|
| `git mv experiments/production_current_truth_rerun.py engine/production_current_truth_rerun.py` | PASS | 保留文件历史。 |
| `uv run --python 3.11 pytest -q -p no:cacheprovider tests/test_cr018_production_current_truth_rerun.py` | `11 passed` | CR018 rerun dry-run contract 行为保持。 |
| `uv run --python 3.11 python -m py_compile engine/production_current_truth_rerun.py tests/test_cr018_production_current_truth_rerun.py` | PASS | 无语法错误。 |
| `git diff --check -- engine/production_current_truth_rerun.py tests/test_cr018_production_current_truth_rerun.py` | PASS | 无 whitespace error。 |
| `rg -n "experiments\.production_current_truth_rerun\|experiments/production_current_truth_rerun\.py" experiments engine tests scripts docs process/changes process/checks` | PASS | 当前源码与测试无旧 import / 旧路径引用；命中仅历史 process 文档和 CR 影响面描述。 |

## Baseline Comparison

新增失败：无。

保持失败：无。

修复的基线失败：未在本切片验证集中确认。

## Decision

Phase 1.2 可以继续进入 contract helper 迁移。该切片只改变 dry-run contract 的归属和导入路径，不改变 S08 payload / QMT admission evidence 语义。
